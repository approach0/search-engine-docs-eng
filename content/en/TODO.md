## TODO
Listed are todo items of this project.

### Demanding
* `\ge` and `\geq` should have the same symbol IDs (there are other similar cases)
* try other UI themes (less colorful maybe?)
* **Field search** (e.g., show/search MSE post tags) with **boolean** query language support (must, should, must-not)
* being able to specify which leaf nodes should avoid substitutions. Maybe a query language like `z = x + y \exact{i}`? 
* support to solve/simplify simple equations/expressions
* monetize this project to survive in this cruel world...
* [✓] initial threshold applied to indexing stage (so that x^2 wouldn't have such a long invlist).
* <del>v3 wildcards support?</del>
* <del>Show informative msg on query TeX parse error.</del>
* [✓] initial threshold, update documentation.
* [✓] recognize query string when no **comma separated**.
* [✓] lower search granularity to sector tree.
* [✓] merge CONST and VAR tokens.
* [✓] different symbol weight: Math token > math variable > sub/sup-script.
* [✓] on-disk math index compression, faster indexer.
* [✓] Re-design representation:
  * [✓] eliminate the impact of sup/subscripts in some cases, e.g., definite and indefinite integrals. 
  * [✓] And also prime variable, e.g., x and x'.
  * [✓] being able to differentiate `$\sum_{i=0}^n x_i = x$` and `$\sum_{i=0} x_i^n = x$`. 
* [✓] put some large resources on CDN (jsdelivr.com)
* [✓] Show last update of index and some visit statistics at homepage.
* [✓] faster TeX rendering using mathjax v3.
* [✓] **Increase cache postlist hit chance** by caching only long posting lists.
* [✓] scalability: Multiple nodes on each core or different machines (using MPI)
* [✓] re-entrant posting list iterators and MNC scoring.
* [✓] **Combined math and text search** under new model.
* [✓] Operand match **highlight**.
* [✓] **Wildcard** under new model.
* [✓] Prefix model efficiency: MaxScore-like **pruning**.
* [✓] Path **operators hashing** to distinguish operator symbols.

### Misc
* [✓] A new mobile-friendly interface
* handle matrix in a better way
* phrase search (specified by quotes)
* photo recognition tool
* faster Chinese tokenizer
* QAC, spelling correction, search suggestion.
* Semantics
  * math equivalence awareness, e.g. 1+1/n = (1+n)/n.
  * **Text synonym** awareness, e.g. horse = pony.
  * good embedding for both text and math, e.g. pythagorean == x^2 + y^2 = z^2

### Additional crawlers
* [✓] StackExchange
  * Math StackExchange
  * MathOverflow
  * CrossValidated
  * physics stackexchange
  * matheducators.stackexchange.com
* [✓] artofproblemsolving.com
* https://ncatlab.org/
* PlanetMath
* Socratic
* https://www.physicsforums.com/
* 悠闲数学娱乐论坛
* 知乎
* <del> Wolfram MathWorld </del> 
* <del> Wikipedia (English) </del> 
* <del>[Proof wiki](https://proofwiki.org/wiki/Main_Page)</del>
* <del> https://brilliant.org/ </del>
* <del> NIST DLMF </del> 
