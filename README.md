# GitHub code navigation

GitHub code navigation helps you to read, navigate, and understand code by showing and linking definitions of a named entity (like a class or method) corresponding to a reference to that entity, as well as references corresponding to an entity's definition. GitHub has developed two code navigation approaches:

* **Search-based:** searches all definitions and references across a repository to find entities with a given name
* **Precise:** resolves definitions and references based on the set of classes, functions, and imported definitions at a given point in your code

Search-based code navigation is implemented using the [Tree-sitter](https://tree-sitter.github.io/tree-sitter/) parser ecosystem. A few languages support precise code navigation, built with [stack graphs](https://github.com/github/stack-graphs).

## Supported languages

Code navigation is supported for the following languages:

| Language         | Search-based       | Precise            |
|------------------|--------------------|--------------------|
| Bash             | :heavy_check_mark: | :x:                |
| C#               | :heavy_check_mark: | :x:                |
| C++              | :heavy_check_mark: | :x:                |
| CodeQL           | :heavy_check_mark: | :x:                |
| Elixir           | :heavy_check_mark: | :x:                |
| Go               | :heavy_check_mark: | :x:                |
| JSX              | :heavy_check_mark: | :x:                |
| Java             | :heavy_check_mark: | :x:                |
| JavaScript       | :heavy_check_mark: | :x:                |
| Lua              | :heavy_check_mark: | :x:                |
| PHP              | :heavy_check_mark: | :x:                |
| Protocol Buffers | :heavy_check_mark: | :x:                |
| Python           | :heavy_check_mark: | :heavy_check_mark: |
| Ruby             | :heavy_check_mark: | :x:                |
| Scala            | :heavy_check_mark: | :x:                |
| Starlark         | :heavy_check_mark: | :x:                |
| Swift            | :heavy_check_mark: | :x:                |
| Typescript       | :heavy_check_mark: | :heavy_check_mark: |


If your programming language is not one of them, you can help us add it.

## Adding code navigation for a new language

To add code navigation for a new language, follow the steps below.

### Linguist

First, the language must be added to [Linguist](https://github.com/github-linguist/linguist).

Linguist is the source of truth for all languages on GitHub. If your language is not included in Linguist, [follow the contribution guidelines](https://github.com/github-linguist/linguist/blob/master/CONTRIBUTING.md#adding-a-language) to get it added.

### Tree-sitter parser

Next, we require a mature Tree-sitter parser for the language. Most popular programming languages already have a Tree-sitter grammar, but if you need to create one, you can [review the documentation for creating a new parser](https://tree-sitter.github.io/tree-sitter/creating-parsers).

### Tags query

Once the language has a Tree-sitter parser, you need to write _tag queries_ to extract the structure of the code for navigation. A tag query is a Scheme-like expression that navigates the Abstract Syntax Tree generated by the Tree-sitter parser to extract a programming language entity. You can look at existing Tree-sitter parsers for inspiration. Parsers usually contain a file called `tags.scm` with tag queries (for example, [see the JavaScript tag queries](https://github.com/tree-sitter/tree-sitter-javascript/blob/master/queries/tags.scm)). Additionally, Tree-sitter has documentation about [using tags queries for code navigation](https://tree-sitter.github.io/tree-sitter/code-navigation-systems).

GitHub code navigation supports extracting definitions for these constructs:

| Category       | Tag                          |
|----------------|------------------------------|
| Class          | `@definition.class`          |
| Constant       | `@definition.constant`       |
| Enum           | `@definition.enum`           |
| Enum variant   | `@definition.enum_variant`   |
| Field          | `@definition.field`          |
| Function       | `@definition.function`       |
| Implementation | `@definition.implementation` |
| Interface      | `@definition.interface`      |
| Macro          | `@definition.macro`          |
| Module         | `@definition.module`         |
| Struct         | `@definition.struct`         |
| Trait          | `@definition.trait`          |
| Type           | `@definition.type`           |
| Union          | `@definition.union`          |

Additionally, references to function or method calls can be extracted as `@reference.call`.

Not all programming languages support all of these constructs. The tag queries should contain only those that make sense for your programming language.

### Fully-qualified names

For languages that support defining functions, methods, or other constructs within another structure, GitHub code navigation supports extracting fully-qualified names.

Here is an example from our Java extractor. Given the following Java code:

```java
public class Cat {
  public String noise() {
    return "meow";
  }
}
```

Our queries to extract `@definition.class` and `@definition.method` and tag the identifiers with `@name`:

```scheme
(class_declaration name: (identifier) @name) @definition.class

(method_declaration name: (identifier) @name) @definition.method
```

The extracted identifier names are used to prefix the method name (`noise`) with its container's name (`Cat`), resulting in the fully-qualified name `Cat::noise`.

However, not all languages define nested items within the container. For example, Go has methods, but they are defined separately from the struct they belong to:

```go
type Cat struct {}

func (c Cat) Noise() string {
    return "meow"
}
```

To implement fully-qualified names for languages like Go, GitHub code navigation adds a `@scope` capture name:

```scheme
(method_declaration
  receiver: (parameter_list (parameter_declaration type: (type_identifier) @scope))
  name: (field_identifier) @name
) @definition.method
```

Our extractor uses the `@scope` capture to create the fully qualified name `Cat.Noise`.

### File a request to add your language

Finally, [create an issue in this repository](https://github.com/github/code-navigation/issues/). We will evaluate adding the parser to the code search indexing system.

We may not add every language. Common reasons to not add a language are an immature Tree-sitter parser, excessive resources required to parse, or low use on GitHub.
