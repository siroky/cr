# Mastering code reviews

Over the past years, I've spent a lot of time doing code reviews. At least that's what GitHub statistics show :). Here are some things I look for in the diffs when checking pull requests and how I approach their resolution. I focus on some areas and patterns based on the frequency and severity of issues that I usually find there. This is not meant to be an exhaustive list, but rather "quality minimum". Note that I spent lot of time at [Mews](https://github.com/MewsSystems/developers) so the stuff here will definitely be biased and opinionated, gravitating towards backend development in C# using functional programming principles.

## Code style and organization

It doesn't matter what the exact code style rules are (e.g. the infamous spaces vs. tabs discussion), what really matters is how consistently they're followed. With consistent code-style, it's easy to spot errors just based on visual "look" of the code. Or by simply noticing that something feels odd. **Consistency leads to error prevention.** Some things you are not able to express in the type system, however consistent follow through on conventions can partially make up for that. That's the main driver, but readability and understandability are important as well.

- If not specified otherwise, follow industry code style standards of the language by default.
- If two things or operations are happening on single line, consider splitting into two lines, utilizing properly named variables. Length of a line is usually good indicator, the longer the line, the higher the chance it does more than it should.
- Instead of complex expressions, prefer a sequence of simple expressions with results in variables. To understand the algorithm, it then usually suffices to read the variable names like a story.
- If methods are doing two things, consider splitting them into multiple methods. Size of a method is usually good indicator, the bigger the method, the higher chance it does too much.
- Prefer "flat" code with little indentation. Depth of indentation is good indicator of places to refactor.
- Prefer "1-dimensional" code that you can just read top down to grasp it. Avoid "2-dimensional" code where you have to read whole line to understand the logic.
- Ordering of anything shouldn't be random, but should follow some logic. When adding a new thing, putting it as the last is almost always bad, rather consider the appropriate position among the other members. Related methods should usually be closer together, overoads should be close together.
- Ordering of method or constructor parameters should be consistent with property declaration. Especially in case of parameters of same type, ordering is very important.
- When structuring a class, order the members according to top-down flow of the algorithm. So that on the top, there would be the high-level algorithm and the more you scroll, the more details you get.
- When adding new parameter or anything in general, always check similar usages in the nearby codebase. And make it consistent, both in terms of naming, typing and ordering.
- When a method has multiple parameters of same type, prefer named arguments in call sites.
- When a method has too many parameters, consider wrapping them into a class.
- Having a "utility" directory usually leads to it becoming a dump. Always try to organize your code and files with some logic / semantics.

## Naming

- Sometimes, convention based naming helps disambiguating meaning of variables of same type. In such case, always check that such convention is followed. E.g. `dateTime` vs. `dateTimeUtc`. Solving on type level is always better though.
- For a single concept, always make sure that a single name is used. Check the related code where such concept is used and use same name.
- Avoid redundancy in naming, always consider how the call site will look like. E.g. instead of `User.GetUserEmail` use `User.GetEmail`.
- Do not embed type names into variable names, always try to express the meaning, not the type / shape.
- If you're introducing some well known or standardized concept, always follow the naming from the standard or RFC where it was published.
- Avoid abbreviations or shortcuts, usually more descriptive name is better. The bigger the scope of the name, the better it should be.
- When you need to have "old" and "new" code side-by-side (for backwards compatibility), always name the new code properly and e.g. prefix the old methods with `Old`. That way, cleanup in future will be easier and will only require just deletion of the old code, no renames.
- Use verbs for method names, use nouns for variable names or properties.
- Try to avoid negation in naming (e.g. `nonSuspendedUsers`), prefer writing the code in "positive" manner (e.g. `activeUsers`) since it's more difficult to juggle with multiple negations in mind.
- With every change, check if name of the method, file and location in directory strucuture makes still sense. Same with namespaces.

## Typing

- Make method parameter types as general as possible to increase reusability.
- Use strong typing (e.g. `NonEmptyList`) to encode requirements on the method parameters instead of runtime checks (`if (list.IsNonEmpty()) { ... }`) inside the method bodies.
- Make method return types as specific as possible to encode guarantees on the result.
- Mutable data types should be avoided, especially in method parameters, return types or class members. Minimize scope of mutability, ideally to local only at most.
- Avoid unintended ambiguous types. E.g. nullable collection. Instead use either non-nullable collection or nullable non-empty collection.
- Consider using enum instead of boolean or instead of limited set of integers (e.g. -1, 0 1).

## Error handling and exceptions

- Fail fast. In case of e.g. contract check, make sure the detection of that is as soon as possible which minimizes the space that needs to be searched for the issue.
- Do not silently swallow errors, e.g. instead of providing subset of result, report an error, push it to the call-site which is usually better suited to handle it.
- Unhandled exceptions should only happen for stuff, that developers can actually fix. Always think of whether occurrence of an exception will trigger a developer to do something about it. If the developer wouldn't be able to do anything, then it should usually be handled otherwise (communicated to end user, retried...).
- Error messages should be as clear as possible, e.g. specifically mentioning what's wrong in the request body instead of just "invalid request".
- Write error messages in the language, tone and level of complexity according to the reader who will likely receive them.

## Performance

The following several things are rather low-hanging fruit, however don't take it too literally. Each of these might be justified. Usually, these are the most common performance issue suspects:

- Is a new loop being introduced? Check how it will perform, how hot the path is and complexity of the body.
- Is a new database query invoked? Check whether it's really needed, cannot be merged with some previous query or avoided otherwise. Check how it will perform and whether it is indexed properly if needed.
- Is there a database query or other IO operation inside loop? Consider converting it into a batch operation if possible or avoid otherwise.
- If there is a performance-sensitive operation being introduced or you're not sure, make sure it is discussed and evaluated.

## Security

- On all application boundaries, ensure access is authenticated and authorized. Especially in APIs and web pages.
- Avoid any sensitive information like access tokens in URLs. Prefer HTTP-only cookies.
- Default values should always lead ot the sctrictest, most secure, most restrictive options.

## Miscellaneous

- Having to pass an identity function as an argument is usually a code-smell and should rather be avoided altogether.
- Avoid duplicate code as much as possible. However, avoid premature abstraction unless, wait for at least two use-cases.
- Avoid redundancy as much as possible, e.g. two method parameters are not needed if second one can always be inferred from the first one.
- Is the code general enough? Take a step back and consider if implementing something more general might be both easier to implement and deliver more value.
- If a new method or function is being added, check surrounding codebase if it could be used there as well.
- Future-proof your code. Think of what will happen in the future if someone else comes in and extends your code naively. Will it lead to errors? Try to prevent this.
- Avoid `TODO` comments or anything actionable inside the codebase. Instead, create issue in the issue tracker corresponding to the action point.
- Is there a comment near the code you're modifying? Always check that it's still valid after your changes.
- Keep number of concepts that can be used to solve a problem minimal. Avoid multiple variants of same thing, reduce "decision paralysis" of others.
- The bigger the scope, the better the code should be in terms of all aspects. During code review, put most emphasis on parts with global scope, e.g. core library.
