# 12\. Workspace Organization¶

## 12.0 Overview¶

The collection of all q entities that have been created in a q session comprises the workspace. This includes not only explicitly created variables but also ancillary items such as enumerations, open handlers, tables mapped from storage and other internal structures.

Like any programming environment of reasonable complexity, q has the potential for name clashes – e.g., when two different scripts define variables of the same name. Because q does not have lexical scoping and local variables are strictly scoped to the function in which they are defined, name clashes between locals is not a problem. But global variables are exposed. To illustrate, suppose you load a script that creates a global `foobar`. Then you load another script that also creates `foobar`. The result is that the second script overwrites the value assigned in the first script.

This is the simplest example of a condition in which two portions of a program contend for the same resource. Things get more complicated when concurrency is introduced, in which case the contention can lead to what is called a _race condition_. There is a school of programming that says mutable shared state is the root of all evil in distributed programming. Fortunately q is single-threaded by default and the implementation of threads (wisely) does not support mutating global variables. Nonetheless, simple name clashes are still a real problem.

## 12.1 Namespaces¶

A partial solution to name clashes is _namespaces_ , which is the idea of placing a (usually hierarchical) structure on names. This is customarily done by specifying a separator character, which is distinct from other valid name characters. You are familiar with this construct from the file system used by most operating systems. The Unix file system uses the separator `/` and the Windows file system uses `\`.

The idea is that a compound name with embedded separators represents a nested series of containers. For example, in the Unix file system, the path `/mydir/myfile` represents a directory (container) `mydir` that holds a file `myfile`. Or `/d1/d2/fn` represents a directory `d1` that contains a directory `d2` that contains the file `fn`. The root container, designated `/` in Unix, holds all other entities.

Namespaces in many programming languages work similarly, using `.` as the separator. In q the containers are called _contexts_. For example, `.jab.x` represents a variable `x` contained in the `jab` context. The context `jab` has the symbolic name ``.jab` and the fully qualified name of its variable is ``.jab.x`.

A notable difference between q namespacing and most others is the way the root is handled. Naming for entities in the root context does not work as you might expect based on the file system analogy. The syntactic form `.name` does **not** refer to the global variable `name` in the root context; rather, it refers to the context `name` in the root. The fully qualified name for an entity `x` in the root is simply `x`.

This has the following consequence that causes grief to qbies.

Namespaces Fact of Life #1

There is no syntactic way inside a function to distinguish between a local variable `x` and the global `x` in the root namespace.

If you follow the Namespacing conventions at the end of this chapter, you can minimize potential discomfort. However, based on real-world experience, we point out the following.

Tip

Inside a function you can not access the global variable `foo` in the root context as `.foo`. Instead, `.foo` refers to the dictionary that holds all entities of the `foo` context. If you assign a value to `.foo` you wipe out the entire `foo` context.

## 12.2 Contexts¶

Namespacing in q is implemented with dictionaries. A context is a dictionary whose keys are the symbolic names of the variables in the corresponding namespace. The context dictionary associates each variable name with its currently assigned value.

The name of a context has the same restrictions as q variable names except that `.` is allowable. The following are all valid names of contexts in the root. 
    
    
```q
.a
.q
.cat
.an_ugly_context_name
```
Thus `.cat.bubba` represents the entity `bubba` in the `.cat` context.

With that out of the way, we can nest contexts.
    
    
```q
.cat.bengal.nuba
.cat.abyssinian.orion
```
Here we have the entity `nuba` in the `bengal` context in the `cat` context; or the `orion` entity in the `abyssinian` context in the `cat` context.

Astute readers already have a question. How do we distinguish between the first case, in which `.cat.bubba` is the variable `bubba` in the `cat` context, and the second case where `.cat.bengal` is a context? For example, can we emulate the file system, which appends the separator `/` to indicate a directory (container). The answer is:

Namespacing Fact of Life #2

The interpretation of a fully qualified name depends on its underlying value. If it is a properly constructed dictionary, it is a context. If it is anything else, it is a variable.

KX reserves all root namespaces of a single letter, as well `.kx`, for its own use. It is worth noting that most q built-in functions that are not written in C live in the `.q` namespace. As of this writing (Sep 2015) the namespaces that are actively used by KX include `.h`, `.j`, `.o`, `.q`, `.u`, `.z` and `.Q`.

Tip

Although q prohibits you from re-assigning its own functions during program execution, some programmers think it is clever to override their definitions in the `q.k` file. This is bad practice bordering on malfeasance. It courts disaster should KX change the internal implementation of those routines in a future release.

## 12.3 Creating Contexts¶

When you start a fresh q session, all global variables you create live in the root context. All previous examples in this tutorial have created global variable variables in the root simply because we did not indicate otherwise – i.e., we used only unqualified variable names.

Start a fresh q session and create some global variables, beginning in the root.
    
    
```q
q)answer:42
q)calculate:{(x*x)+(-1*x)+1}
```
To use namespacing, there is no equivalent to `mkdir` that creates a context before it can be used. Simply specify a qualified name and the context will be created if it does not exist.
    
    
```q
q).foo.a:42 / context foo created
q).foo.double:{2*x} / existing context foo used
```
Now suppose a separate script uses the `bar` namespace with different definitions for `a` and `double`.
    
    
```q
q).bar.a:43 / context bar created
q).bar.double:{2*x} / existing context bar used
```
We can now load that script without clobbering our root variables, so we have solved the original problem posed in the introduction to this chapter.

Be very careful about placing globals in the root. As the author’s father would say, “You’re cruisin’ for a bruisin’”.

When working with contexts, apply an overload of the key operator to list the names of all contexts in the root.
    
    
```q
q)key `
`q`Q`h`j`o`foo`bar
```
There is no need to stop at one level of namespace. You can nest as deeply as you like and all the intermediate contexts will be created as needed. Suppose we work for a company Adiabatic Programming Laboratory. We might structure our library of computation routines something like the following.
    
    
```q
q).apl.core.calc.array.fill0:{0.0^x}
```
This single assignment causes the nested series of contexts `apl`, `core`, `calc` and `array` to be created and a function `fill0` to be placed in the innermost context `array`.

## 12.4 Context as Dictionary¶

As mentioned previously, a context is a specially formatted q dictionary. Its keys are the symbolic names of the variables it contains; each is associated with the current value of the corresponding variable.

The context dictionaries are dictionaries that live in the workspace along with ordinary variables, which have special meaning to q. We can perform operations on them just as with our own dictionaries.

When we start a fresh q session, the root dictionary is empty. As with any variable, we can reveal the root context by applying `get` (or `value`) to its name and then applying the utility [`.Q.s1`](https://code.kx.com/q/ref/dotq/#qs1-string-representation) to display its internal form since it is empty.
    
    
```q
q).Q.s1  get `.
"(`symbol$())!()"
```
Here we see an empty key list of symbols and a general empty list of values. After defining variables in the root, an ordinary dictionary emerges.
    
    
```q
q)get `.
a     | 42
double| {2*x}
```
The entries in a context dictionary are sorted by name.

Things get more interesting when we use namespacing and create a context.
    
    
```q
q).jab.wrong:43
q)get `.
a     | 42
double| {2*x}
q)get `.jab
      | ::
wrong | 43
```
Observe that the newly created context dictionary for `.jab` is **not** physically in the root directory; it is a separate dictionary.

Advanced

You might wonder about the significance of the `::` in the first entry of the `.jab` context dictionary. It is there to prevent the value list of the context dictionary from collapsing to a simple list in case all the variables should have values of the same scalar type. This would then prevent subsequent definition of variables of other types in that context.

As they say in certain quarters, q eats its own dog food, meaning all ordinary operations can be performed on a context dictionary. For example, we can look up the values associated with ``a` in the root and with ``wrong` in the `.jab` context.
    
    
```q
q)`.[`a]
42
q)`.jab[`wrong]
43
```
In fact you can use this to access an obscured global inside a function.
    
    
```q
q)a
42
q){a:43; `.[`a]}[]
42
```
While this sort of thing makes for cute parlor tricks, we do not advise using it in real code.

Use `get` and `set` to retrieve and store global variables by name, especially inside functions.
    
    
```q
 q){`a set 1+ get `a}[]
 `a
 q)43
 43
```
## 12.5 Expunging from a Context¶

We have seen that a context is a directory that maps a context’s entity names to their values. This means that in order to expunge an entity from a context, we could directly remove it from the dictionary. But this is bad practice since it uses the underlying implementation and the actual expression is prone to typing error.

Instead, q provides a special overload of the delete template explicitly for this purpose.
    
    
```q
q)a:42
q).jab.wrong:43
q)delete a from `.
`.
q)a
'a
q)delete wrong from `.jab
`.jab
q)\v
`symbol$()
q)\v .jab
`symbol$()
```
Indeed, we issue the `\v` command to display the names of variables in a context (current context if none is specified) to verify that the variables are gone.

## 12.6 Saving and Loading Contexts¶

Since a context is a dictionary, you can persist it – and all its contents – as a serialized q object using `set`.

For example, to write out the root context, retrieve it by name and set it into a file.
    
    
```q
q)`:/data/root set get `.
`:/data/root
```
Conversely, you can read the serialized object back into memory using `get` on the file and then overwrite the current root context using `set`
    
    
```q
q)`. set get `:/data/root
```
If you organize your globals carefully, this makes check-pointing the state of your application very simple. Overlaying the root context replaces all its entries. As the saying goes, “Be careful what you ask for.”

## 12.7 Working in a Context¶

We return to the analogy of paths in the file system. The operating system keeps track of the current working directory for each program. This is merely a bit of state that points to a logical location in the file system. The main feature is it allows paths for items at or below the current working directory to be specified in _relative_ form.

For example, if we start with the current working directory as the root and we wish to access a file `/db/2015.03.01/trades.dat`, then we must use the full path. However, if we issue the OS command,
    
    
```q
cd /db
```
then `/db` becomes the current working directory and we can specify the path of the file as simply `2015.03.01/trades.dat`. Note the absence of the leading `/`, which signifies a relative path as opposed to an absolute one. This provides significant simplification for deeply nested directory structures, which often arises in large development environments.

A q session has a similar concept. At any point in the session there is a _current working context_. At the beginning of a fresh q session, the current working context is the root context and you use absolute names for all global variables. We can change the working context with the `\d` command (for “directory” by analogy to the file system). After you change the current working context, you can use relative names for global variables. Here we create two globals in the root context and then switch the current working context to show relative names.
    
    
```q
q).jab.util.counter:0
q).jab.util.incrctr:{[] `.jab.util.counter set 1+get `.jab.util.counter}
q).jab.util.incrctr[]
`.jab.util.counter
q)\d .jab
q.jab)util.incrctr[]
`.jab.util.counter
q.jab)util.counter
2
```
Notice that the q prompt changes to let you know the current working context.

There is a subtle point about how globals are bound during function definition that we must point out. To illustrate, start a fresh q session and do the following.
    
    
```q
q)state:`NY
q).jab.f1:{[] state}
q)\d .jab
q.jab)state:0N
q.jab)f2:{[] state}
q.jab)\d .
q).jab.f1[]
`NY
q).jab.f2[]
0N
q).jab.f1
{[] state}
q).jab.f2
{[] state}
```
We began by defining a variable `state` in the root context to hold a two-character code for one of the fifty United States of America. Also from the root context, we define a function `f1` in the `.jab` context that returns the value of `state` from the root context. Next we switch to the `.jab` context and define a variable `state` that holds the integer value of a state machine, along with a function `f2` that returns the value of this global.

We return to the root context, apply each function using an absolute path name and observe the expected values. Namespacing has done its job by avoiding a clash between the disparate state variables, so what’s the issue? Look closely at the display of the two function bodies. They are identical! How does q know which `state` variable to access at runtime?

Namespacing Fact of Life #3

Any unqualified global reference in a function is bound to the current working context in effect at the point of the function **definition**. That context is not explicitly displayed in the function body.

This has the undesirable consequence of requiring a potential user of the function to know the circumstances of its definition.

But wait, there’s more. Let’s redo the initial example of this section in a fresh q session, but change the current working context two levels down instead of one.
    
    
```q
q).jab.util.counter:0
q).jab.util.incrctr:{[] `.jab.util.counter set 1+get `.jab.util.counter}
q)\d .jab.util
'.jab.util
```
You can’t do it!

Namespacing Fact of Life #4

You can only set the current working context one level down from the root. This is truly unfortunate.

You might be tempted to abandon namespacing altogether, but don’t. If you adhere to the following recommendations, you will avoid the issues mentioned above and namespacing will work effectively.

**Recommendations for Namespacing** :

  * Use namespacing at arbitrary depth to create hierarchical organization of global variables, both data and functions.
  * Keep related functions and any globals they require together in an appropriate context in the hierarchy with a descriptive name.
  * By convention, q namespaces are all lower case.
  * Define all global entities from the root context using fully qualified names.
  * Always refer to global entities using fully qualified names.



Essentially the last two recommendations amount to avoiding all use of [`\d`](https://code.kx.com/q/basics/syscmds/#d-directory "Directory") to switch the current working context. Here is one way to define the functions `f1` and `f2` above in a fresh q session so that everything is explicit.
    
    
```q
q).jab.geog.state:`NY
q).jab.geog.f1:{[] .jab.geog.state}
q).jab.machine.state:0N
q).jab.machine.f2:{[] .jab.machine.state}
q).jab.geog.f1
{[] .jab.geog.state}
q).jab.machine.f2
{[] .jab.machine.state}
```
Back to top 
