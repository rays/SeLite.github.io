---
layout: default
---
{% include links %}
* TOC
{:toc}

# Summary #
This describes and compares ways of test automation and its data. See also  [Terminology](Terminology) and pictures at [TestMethods](TestMethods).

# SeLite testing of DB-driven web apps #
It is testing of DB-driven applications, where [scripts][script] keep and update a copy of the application's DB. The scripts themselves (the steps, conditional logic etc.) are not necessarily in a DB. The script-specific input data doesn't have to be in a DB (but it may be).

## The goal ##
SeLite is for thorough, yet practical testing of DB-driven apps. Your tests

  * can validate screens based on (a replica of) all data, rather than based on just recent actions that only modify a part of data
  * don't need to specifically track the history of actions from previous test sessions
  * have to update their own DB to reflect those actions
    * where needed (they can skip changes done by the application to fields not used by the test - e.g. history logs)
  * need to be repeatable and robust
    * i.e. not specific to a test session or to a browser's session
    * the data is flexible and not trivial, e.g.
      * based on the [script DB], rather than just hard-coded or sequential
      * random, semi random or partially random
    * actions/groups of actions may be done in random order
For example, you don't need SeLite  if your test just
  * creates a new record
  * navigates to/locates the record somehow (possibly by creation time)
  * validates that the record reflects what the test just has done to it
  * ignores the rest (other records, totals or averages etc.)
because that
  * doesn't need data from other records, and
  * it won't need the data of this new record, when it's re-run next time.
But you'll benefit from SeLite if you continually test the application, when you need the test to incorporate effect of actions done by all previous testing.

# Types of automated testing of DB-driven web applications #
This is not Selenium IDE or SeLite-specific, but it only covers what can be done with Selenium. There are a few options, the easiest first:

## 1. Test uses no DB ##
This is hardly sufficient as thorough testing. The [app DB]

  * has initial state 'frozen' to make the test work
  * needs to be reset to that initial state (every time the test is run)

The test

  * has no own DB
  * has no access to [app DB] (except for access to reset it - if needed and automated)
  * depends on the initial [app DB] to be in the expected state
  * enters/modifies the app data
    * by navigating the app via its UI
    * based on fixed/random/partially random data set
    * stores the data in test session (if needed)
    * validates the data presented by the app UI (against that session data or against the fixed data)
  * doesn't know about any previously entered/modified data present in [app DB], if (re)started

Maintenance cost makes this not feasible for long term. Its parts could be reused with SeLite. However, you may want to use different locators, depending on the data schema, page navigation, login functionality. That may be easier when starting from scratch.

## 2. Web app and its test use the same DB ##
Like #1, but the test reads from [app DB], which

  * has no 'frozen' initial state
  * may need resetting/partial resetting from time to time

The test

  * has no own DB
  * has read-only access (back door) to [app DB] (and access to reset it - if needed and automated)
  * generates a fixed/random/controlled random data set, which it enters/modifies via the app UI
  * it validates the data presented by the app UI against
    * the app data (reloaded from the app DB)
      * possibly not detecting hidden/silent application errors that
        * cause app to save incorrect/incomplete/inconsistent information
        * pass unnoticed (unless this errornous data gets detected later but still in the same test session)
    * the test session
      * possibly detecting hidden errors, but it
        * is difficult to implement, if an error shows up only several steps/stages after its initial cause
        * can't identify errors caused by previous test runs (not tracked in the current test session)
  * if (re)started, it gets the initial state based on the current [app DB] only, with no other track of the changes caused by previous test runs

Single source of truth - i.e. same DB used by the app and by the test - causes hidden problems (false positives). The test may succeed, but the data flow has bugs.

The hidden data error may be detected later, but only if

  * there is a way to determine (re-calculate...) the correct value from the rest of the data and
  * the application presents the rest of the data needed for this (it may include pagination over the records...) and
  * the tester thinks of comparing the actual value and the expected value.
But, it's difficult to creates tests to cover this. So it would involve human validation.

Even worse, the data field (or fields) affected by the hidden bug may be not be verifiable by the rest of the data at all. That's usually when the DB schema doesn't cover the history of actions/transactions, or it removes old actions/transactions and it doesn't cover the intermediary steps. For example, a banking application could keep only the current amount and transaction amounts (credits/debits) for a fixed period, but it doesn't include amounts current at those transaction dates. If there's an error in applying a certain type of transaction and it results in an incorrect update of current amount, there's no way to detect this error - unless you have a separate copy of the data.

See [simple online demo](http://htmlpreview.github.io/?https://github.com/selite/selite/blob/master/demo/simple/web/index.html) as an example.

## 3. Web app and its test have separate DBs ##
This explains [Overview](./) > [Advantages of test data separation](./#advantages-of-test-data-separation). It's like #2, but the test

  * has its own DB
    * which is initially a copy of [app DB], with
      * schema
        * either exactly same as in [script DB], or
        * modified but logically equivalent to that of [script DB], or
        * subset of [script DB] schema or its simplified version (as relevant to testing)
      * data
        * containing all data from app DB, or
        * being a narrowed subset of app DB data
          * then the test needs to be aware that the app may show more entries than what is in [script DB], i.e. the test needs to filter/scroll/navigate across the page(s) of records shown by the app, to locate the records that it wants to test
          * with command `insertCaptureKey` in DB Objects > Selenese reference
          * see `RecordSetHolder.prototype.select` in DbObjects.js
          * see `narrowBy` and `alwaysTestGeneratingKeys` in common settings
          * If `narrowBy` is set, then 
          * @TODO move to GeneralFramework.md: SeLiteData.Db instance (and optionally SeLiteData.Table instances) have `narrowMethod` field. It indicates a method of narrowing. Default is by prefix.
          * SeLiteData.Table instances have fields `narrowColumn`. That's the name of the column, which will be automatically used for narrowing. DbStorage will inject `narrowBy` value (if any) to any new records. (There's also an optional `narrowMaxWidth` to limit the used part of `narrowBy` value.) When matching the formulas, they will also filter by `narrowBy` (if any).
          * If `alwaysTestGeneratingKeys` is `true`, then the
        * should be approximately equal to the app data (for approximate fields see [HandlingData](HandlingData) and [TimeStamps](TimeStamps))
        * supported by UUID and UUID hashes
  * keeps its DB in sync with [app DB] (or its part), updating [script DB] to reflect changes in app DB
    * but it doesn't blindly copy/replicate the changes from app DB to script DB (via a back door), since that would effectively be approach #2
    * the test updates its DB on its own, but in a way that the tester believes the app updates its DB
    * ideally the test doesn't use exactly same SQL queries as the app, but ones that are logically equivalent
      * this helps to detect SQL-level logical bugs (other than syntax errors)
      * SeLite helps with this, by providing an object oriented layer (which is unlikely to be used by the web app, therefore the app uses a different method to generate SQL, so there's a higher chance to detect an error in the app)
  * validates the data presented by the app UI against the data in [script DB], which enables the test to detect hidden/silent application errors
    * whether during the test run when the error happens, or during a later test
  * [script DB] may get out of sync if
    * the test is incomplete/incorrect (being developed), or
    * the app and/or test dies/times out
    * [app DB] or [script DB] is changed in a way other than through the test (likely during development of the test or the app)
  * and then
    * [script DB] needs to be reloaded from new a copy of [app DB] (if healthy), or
    * both script and app DBs needs to be reset
    * potential challenge: automate this DB reload/reset

If the same error exists in both the app and the test, then it doesn't get detected automatically. That may be at various levels

  * DB schema
    * because [app DB] and [script DB] schema are same/equivalent
    * there's no big need to detect these errors by automated testing, since the schema
      * is the cornerstone of applications/systems, so it's usually well defined, checked and tested
      * doesn't change as much as functionality, therefore there's a less chance of human error
      * is often used by multiple applications, so such an error will be noticed by a person sooner than an error in an application (in general)
  * DB queries
    * if using same queries/patterns for the app and the test; avoid this by using SeLite DataObjects and formulas to generate queries instead
    * if using SQLite for the app (not common for server-side web apps)
  * business/presentation logic - that's focus of SeLite
    * as a precaution, have corresponding parts of the app and the test implemented differently, e.g. developed
      * by different people, and/or
      * with a time period between implementing them, and/or
      * developing the test first and only then the app functionality, and/or
      * using different technique
        * this is implied for DB/business layer, since SeLite test components are in Javascript (while a server-side app is usually not in Javascript)
        * your test can use DbObjects extension (e.g. class DbRecordSetFormula) and write less custom SQL queries (no need to reinvent the wheel)

SeLite allows us to do this. See also [HandlingData](HandlingData).

# SQLite side effects of SeLite #
SQLite follows SQL standard pretty well, but it may differ to your application's RDBMS. That can cause false negative errors. See [SQLiteSpecifics](SQLiteSpecifics).