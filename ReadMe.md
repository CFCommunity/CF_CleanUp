# CF_CleanUp

This document is intended to be a collection of recommendations to Adobe from the ColdFusion Community (meaning all ColdFusion developers), on how to improve the CFML and CFScript languages.

## WAT?

ColdFusion has seen continuous changes for going on 11 major versions over the course of the last 18+ years. Unfortunately we haven't always taken time to clean house. Being the first platform to do something means that sometimes your implementation isn't the best. There's a little bit of a mess, and it's time to clean it up. This is your chance to point out a mess you're familiar with and propose a method of better organization.

_Fork, add your two cents, and submit a pull request. No change is too small!_

## Opposing Viewpoints

There are bound to be disagreements on proposals, and we welcome differing opinions, and would be more than happy to have multiple proposals to fix the same problem. _Bring it on._

## Organization

Add proposals of specific changes to a tag or function's implementation to the alphabetized tag/function **Implementation Changes** section.

Add proposals for function/tag renaming, moving, etc to the **Organizational Changes** section.

## Backwards Compatibility

Some of the proposed changes are drastic, and would break a lot of backward compatibility, but _we have to stop thinking of this as a bad thing_. You can't make an omelet without cracking a few eggs.

**Permanent deprecation is also a bad thing.** There have been many things deprecated in CF's history, but it's rare that something is actually removed; Verity not withstanding (but that was a monetarily motivated decision). _It is our hope that deprecations that come as a result of these proposals include a strict EOL._ Promise to support both the old way and the new way for no more than one or two Major versions (e.g. CF 11) but with the next version completely remove support for the deprecated features.

Think of it this way: If you only ever sweep dirt into the corner of your house, the house will never be clean. You've got to pick it up and throw it out at some point.

## Proposals

### Organizational Changes

#### Headless Functions

We have too many headless functions. Instead of 24 headless `ArrayX()` functions, we should have ~24 `arrayObj.x()` methods. The same goes for structures, strings, etc.

 * ArrayAppend
 * ArrayAvg
 * ArrayClear
 * ArrayDeleteAt
 * ArrayInsertAt
 * ArrayContains
 * ArrayIsDefined
 * ArrayIsEmpty
 * ArrayLen
 * ArrayMax
 * ArrayMin
 * ArrayDelete
 * ArrayNew
 * ArrayPrepend
 * ArrayResize
 * ArraySet
 * ArraySort
 * ArrayFind
 * ArrayFindNoCase
 * ArraySum
 * ArraySwap
 * ArrayToList
 * IsArray
 * ListToArray

Perhaps `myArray.length` would be better implemented as a property than a method. Likewise, some may not be good fits for instance methods, like `IsArray()`. In this case, we should either leave the functions as headless, or create a sort of class object from which they can be used without an object instance, such as: `Array.isArray( foo )`, or `Array.new(2)`.

Headless functions are not bad by definition, but making **all** functions headless is a bad choice.

### Implementation Changes

#### CFML

##### CFQuery

Inserts that result in an auto-generated Identity column value should all return the created value in the same result key name. Currently every database platform has its own key (e.g. IDENTITYCOL for MSSQL, and GENERATED_KEY for MySQL), which is problematic for portable software (think blogs, frameworks, etc) as they have to have switches to deal with all possible cases. If the key was always the same name regardless of the DB platform, it would be easier for all developers to work with, and projects would be more portable. See also, [Bug #3490074](https://bugbase.adobe.com/index.cfm?event=bug&id=3490074).

#### CFScript

##### Queries

The current problem with CFScript queries is that the syntax is awkward and they don't behave the same as their tag counterparts. For example, to do a Query of Queries, you've got to pass the existing Query resultset into the new Query object so that it can be referenced. These are symptoms of a larger problem: While implementing some tags as CFC's is a fine stop-gap, Queries are simply not a good fit.

Our proposal is to remove Query.cfc and properly add it to the language. Our proposed syntax would be:

    [result =] query( SQL, parameters, options );

    parameters: {id: 1, lastName: 'foo'}
    options: {cachedWithin: createTimespan(...), ...}

Example:

    result = query(
        "insert into myTable (name) values (:name)",
        { name: "Bob the Builder" },
        { datasource: "myDb", username: "db_user", password: "db_pass" }
    );
    inserted_id = result.IdentityCol;

Being properly implemented into the CFScript language, instead of deferring to CFCs, should also mean that this syntax would properly have access to available-scoped query objects:

    variables.people = query("select name, age from person");

    variables.octogenarians = query("
        select name, age from variables.people
        where age >= 80 and age <= 89
    ", {}, { dbType: "query" });

We believe that this syntax is easy to remember, terse, and just as intuitive as the `<cfquery>` tag. There is no need to call `execute()`, and the result is always returned.
