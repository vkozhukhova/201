# Spark Catalyst

## Catalyst features

* based on functional programming constructs in Scala. 
* design had two purposes. 
  * make it easy to add new optimization techniques and features to Spark SQL, especially for the purpose of tackling various problems with big data \(e.g., semistructured data and advanced analytics\). 
  * enable external developers to extend the optimizer — for example, by adding data source specific rules that can push filtering or aggregation into external storage systems, or support for new data types. 
* supports both rule-based and cost-based optimization.
* contains a general library for representing trees and applying rules to manipulate them. 
* On top of this framework - libraries specific to relational query processing \(e.g., expressions, logical query plans\), and several sets of rules that handle different phases of query execution: analysis, logical optimization, physical planning, and code generation to compile parts of queries to Java bytecode. 
* use Scala feature, quasiquotes, that makes it easy to generate code at runtime from composable expressions. 
* offers several public extension points, including external data sources and user-defined types.

## Trees

* main data type in Catalyst - a tree composed of node objects. 
* Each node has a node type and zero or more children. 
* New node types are defined in Scala as subclasses of the TreeNode class. 
* These objects are immutable and can be manipulated using functional transformations

three node classes can be used to build up trees:

* `Literal(value: Int)`: a constant value
* `Attribute(name: String)`: an attribute from an input row, e.g.,“x”
* `Add(left: TreeNode, right: TreeNode)`: sum of two expressions.

tree for the expression `x+(1+2)`:

```text
Add(Attribute(x), Add(Literal(1), Literal(2)))
```

## Rules

* Trees can be manipulated using **rules - functions from a tree to another tree**. 
* rule can run arbitrary code on its input tree \(given that this tree is just a Scala object\), 
* the most common approach - use a set of pattern matching functions that find and replace subtrees with a specific structure.
* Pattern matching allows extracting values from potentially nested structures of algebraic data types. 
* trees offer a transform method that applies a pattern matching function recursively on all nodes of the tree, transforming the ones that match each pattern to a result.

e.g.

```text
tree.transform {
  case Add(Literal(c1), Literal(c2)) => Literal(c1+c2)
}
```

Apply to tree `x+(1+2)` -&gt; new tree `x+3`

* pattern matching expression that is passed to transform is a partial function: it only needs to match to a subset of all possible input trees. 
* Catalyst will test which parts of a tree a given rule applies to, automatically skipping over and descending into subtrees that do not match. 
* rules only need to reason about the trees where a given optimization applies and not those that do not match. 
* rules do not need to be modified as new types of operators are added to the system.
* Rules can match multiple patterns in the same transform call, making it very concise to implement multiple transformations at once
* rules may need to execute multiple times to fully transform a tree. 
* Catalyst groups rules into batches, and executes each batch until it reaches a fixed point - until the tree stops changing after applying its rules. 
* Running rules to fixed point means that each rule can be simple and self-contained, and yet still eventually have larger global effects on a tree. 
* rule conditions and their bodies can contain arbitrary Scala code. This gives Catalyst more power than domain specific languages for optimizers, while keeping it concise for simple rules.

## Using Catalyst in Spark SQL

four phases

* analyzing a logical plan to resolve references - purely rule-based 
* logical plan optimization - purely rule-based
* physical planning: Catalyst may generate multiple plans and compare them based on cost
* code generation to compile parts of the query to Java bytecode - purely rule-based
* Each phase uses different types of tree nodes;
* Catalyst includes libraries of nodes for expressions, data types, and logical and physical operators. 

### Analysis

* begins with a relation to be computed, either from an abstract syntax tree \(AST\) returned by a SQL parser, or from a DataFrame object constructed using the API. 
* the relation may contain unresolved attribute references or relations: e.g. in query `SELECT col FROM sales`, the type of col or whether it is a valid column name is not known until we look up the table sales. 
* An attribute is called unresolved if we do not know its type or have not matched it to an input table \(or an alias\). 
* Spark SQL uses Catalyst rules and a Catalog object that tracks the tables in all data sources to resolve these attributes. 
* It starts by building an “unresolved logical plan” tree with unbound attributes and data types, then applies rules that do the following:
  * Looking up relations by name from the catalog.
  * Mapping named attributes, such as col, to the input provided given operator’s children.
  * Determining which attributes refer to the same value to give them a unique ID \(which later allows optimization of expressions such as `col = col`\).
  * Propagating and coercing types through expressions: for example, we cannot know the return type of `1 + col` until we have resolved col and possibly casted its subexpressions to a compatible types.

### Logical Optimizations

* applies standard rule-based optimizations to the logical plan. 
* rule-based optimizations: 
  * constant folding, 
  * predicate pushdown, 
  * projection pruning, 
  * null propagation, 
  * Boolean expression simplification, etc. 
* it's simple to add rules for a wide variety of situations.

### Physical Planning

* Cost-based optimization is performed by generating multiple plans using rules, and then computing their costs. 
* Spark SQL takes a logical plan and generates one or more physical plans, using physical operators that match the Spark execution engine. 
* It then selects a plan using a cost model. 
* cost-based optimization is only used to select join algorithms: for relations that are known to be small, Spark SQL uses a broadcast join, using a peer-to-peer broadcast facility available in Spark. 
* The framework supports broader use of cost-based optimization, however, as costs can be estimated recursively for a whole tree using a rule. 
* physical planner also performs rule-based physical optimizations, such as pipelining projections or filters into one Spark map operation. 
* it can push operations from the logical plan into data sources that support predicate or projection pushdown. 

### Code Generation

* involves generating Java bytecode to run on each machine. 
* Because Spark SQL often operates on in-memory datasets, where processing is CPU-bound, we wanted to support code generation to speed up execution. 
* Catalyst relies on a special feature of the Scala language, quasiquotes, to make code generation simpler. 
* Quasiquotes allow the programmatic construction of abstract syntax trees \(ASTs\) in the Scala language, which can then be fed to the Scala compiler at runtime to generate bytecode. 
* We use Catalyst to transform a tree representing an expression in SQL to an AST for Scala code to evaluate that expression, and then compile and run the generated code.

a function to translate a specific expression tree to a Scala AST :

```text
def compile(node: Node): AST = node match {
  case Literal(value) => q"$value"
  case Attribute(name) => q"row.get($name)"
  case Add(left, right) => q"${compile(left)} + ${compile(right)}"
}
```

* strings beginning with `q` are quasiquotes, meaning that although they look like strings, they are parsed by the Scala compiler at compile time and represent ASTs for the code within. 
* Quasiquotes can have variables or other ASTs spliced into them, indicated using `$` notation. 
* `Literal(1)` would become the Scala AST for 1, while `Attribute("x")` becomes `row.get("x")`. In the end, a tree like `Add(Literal(1), Attribute("x"))` becomes an AST for a Scala expression like `1+row.get("x")`.

#### Quasiquotes

* are type-checked at compile time to ensure that only appropriate ASTs or literals are substituted in, making them significantly more useable than string concatenation, and they result directly in a Scala AST instead of running the Scala parser at runtime. 
* are highly composable, as the code generation rule for each node does not need to know how the trees returned by its children are constructed. 
* the resulting code is further optimized by the Scala compiler in case there are expression-level optimizations that Catalyst missed. - they let us generate code with performance similar to hand-tuned programs.
* also work well with running on native Java objects: when accessing fields from these objects, we can code-generate a direct access to the required field, instead of having to copy the object into a Spark SQL Row and use the Row’s accessor methods. 
* can combine code-generated evaluation with interpreted evaluation for expressions we do not yet generate code for, since the Scala code we compile can directly call into our expression interpreter.

