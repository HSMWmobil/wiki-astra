# Testing

Testing is indispensable in order to easily check whether new changes might break existing 
logical components of the app. Therefore, we highly encourage testing newly developed components 
as well as to cover new cases in existing tests.

## Unit Tests

All logical components should be tested through _unit tests_.
Unit testing means that we want to encapsulate a single logical functionality and mock all 
dependencies so we can check whether the function (therefore, the single unit) works in isolation.

There are a few rules you should follow when writing tests:
- Create a separate test file for each class file under the folder `/test`. 
- The file path should exactly match the file path of the original Dart class file and have the 
  prefix `_test`.
- If a Dart file has multiple classes, make sure to wrap each class into a `group()` function, 
  containing the class name.
- Use `setUp()` to instantiate mocks that are needed in every test, but as a rule of thumb 
  define the mocking behavior in the test case itself.

Mocking means simulation correctly working dependencies our unit might need.
For mocking in this app, we use [mocktail](https://pub.dev/packages/mocktail).
Therefore, creating a class usable as a mock is easy: Just extend `Mock` and implement the class 
base interface, like shown here for the example of a repository implementation:
```dart
  class MockAuthRepository extends Mock implements AuthRepository {}
```

## Integration Tests

Coming soon...

## Widget Tests

Coming soon...
