---
description: >-
  In-depth exploration, discover how to harness the power of Flutter's testing capabilities to test your PRs for AppFlowy. Get a better chance of getting your PRs merged with well-written tests.
---

# 🧪 Mastering Flutter Testing in AppFlowy

by Mayur, as part of the AppFlowy Mentorship Program

## Introduction

AppFlowy, our favorite Open Source Knowledge Management Tool, is a rapidly evolving software powered by a genius team of maintainers and passionate open-source contributors. This rapid speed of development is matched with rigorous testing, to ensure top-most quality and seamless user experience.

So if you are looking to contribute to AppFlowy, then you must harness the power of Software Testing.
Testing serves as a critical safeguard against regressions and bugs, allowing developers to catch issues early in the development cycle, saving time and resources in the long run. 

Moreover, it fosters a culture of confidence in code modifications, enabling maintainers and contributors to make necessary enhancements and changes without fear of breaking existing functionality. According to me, this is the most crucial advantage of testing. 

To a maintainer at AppFlowy, testing provides the confidence that your new Pull Request is safe to be merged. Also, you write tests to ensure that someone else might not break the feature you introduced, this is done by your written tests for your PRs.

Now that we understand the importance of Testing, let's explore how to actually do it. In this article, we will talk about how to test with Flutter. We shall learn different types of testing that you can do in AppFlowy. So let's begin.

## The A, B, C of Testing in Flutter

Before we write tests, let us remind ourselves what is a good software test. 
 * A good software test is a program that verifies the intended behavior of your code. 
 * It must have a clear purpose and it should test a specific behavior. 
 * Generally, there should be only a single reason behind the failing test. 

While there are many types of tests possible in Flutter, we shall constrain our discussion to the following types. These are the most common tests that the maintainers at AppFlowy expect you to write:
 * Unit Tests
 * Bloc Tests
 * Widget Tests
 * Integration Tests

Let us look at the purpose of the above-mentioned types of testing by reviewing some tests written in AppFlowy.

## Unit Tests

The following are characteristics of Unit Tests:
 * used for verifying the expected behavior of a single Class or Method. 
 * smaller than other tests 
 * faster to execute. 
 * used to test the functionality of a Class in isolation without other dependencies

To write your unit tests, you will need to create a new file within the directory: 

```
frontend\appflowy_flutter\test\unit_tests
```

Let's take a look at some examples: 

Our first example is created by Alex, this example tests a method called levenshtein(). 
This method is used in the Font Family Selection setting in AppFlowy. It is used to calculate the distance between two strings, where less distance implies that strings are identical. 

```dart
// appflowy_flutter\test\unit_test\algorithm\levenshtein_test.dart
import 'package:appflowy/workspace/presentation/settings/widgets/settings_appearance/levenshtein.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('Levenshtein distance between identical strings', () {
    final distance = levenshtein('abc', 'abc');
    expect(distance, 0);
  });

  test('Levenshtein distance between strings of different lengths', () {
    final distance = levenshtein('kitten', 'sitting');
    expect(distance, 3);
  });

  test('Levenshtein distance between case-insensitive strings', () {
    final distance = levenshtein('Hello', 'hello', caseSensitive: false);
    expect(distance, 0);
  });
  
  //more tests ...
}
```

Like any other dart program, our unit test has a `main()` method. Immediately inside the main method, we can see a `test()` method. This method contains a test description, which describes what the test aims to verify. 

Inside the `test()` we call the method under test and pass it some parameters. But I want to draw your attention to the line below it. The line with the `expect()`. This method takes in two parameters, the first is the actual value, and the second is the expected value. If the actual value matches the expected value, we say our test passed successfully. 

Inspired by the existing tests I have seen in AppFlowy I wrote a test for my feature of Customizable Shortcuts. This time we are testing a class called `SettingsShortcutService`: 

```dart
// appflowy_flutter\test\unit_test\settings\shortcuts\settings_shortcut_service_test.dart
void main() {
  late SettingsShortcutService service;
  late File mockFile;
  String shortcutsJson = '';

  setUp(() async {
    final MemoryFileSystem fileSystem = MemoryFileSystem.test();
    mockFile = await fileSystem.file("shortcuts.json").create(recursive: true);
    service = SettingsShortcutService(file: mockFile);
    shortcutsJson = //some json map, see in original code
  });

  group("Settings Shortcut Service", () {
    test(
      "returns default standard shortcuts if file is empty",
      () async {
        expect(await service.getCustomizeShortcuts(), []);
      },
    );

    test('returns updated shortcut event list from json', () {
      final commandShortcuts = service.getShortcutsFromJson(shortcutsJson);
      final cursorUpShortcut = commandShortcuts
          .firstWhere((el) => el.key == "move the cuthe rsor upward");
      final cursorDownShortcut = commandShortcuts
          .firstWhere((el) => el.key == "move the cursor downward");
          
      expect(commandShortcuts.length,3);
      expect(cursorUpShortcut.command, "alt+arrow up");
      expect(cursorDownShortcut.command, "alt+arrow down");
    });
  });
 }
```

This service is responsible for saving the shortcuts list to a file and retrieving these shortcuts from the file when required. 

This test introduces some new concepts we need while writing unit tests. Observe how we are calling a `setUp()` method to set up a mock file for testing. This file is actually a dependency for the class we are testing. 

In tests that you are going to write, you may want to initialize some dependencies before testing, this is done inside the `setUp()` and `setUpAll()` methods. The difference between the `setUp()` method and `setUpAll()` is 

> setUp is used for per-test setup, ensuring each test case starts with a clean slate, 
> while setUpAll is used for one-time setup that is shared across all test cases in a suite.

Next, we have the `group()` method that surrounds our tests. Groups are used to well ... group tests together. Typically while testing classes, you want to separate a bunch of tests from another bunch of tests because they might have a characteristic difference, even when they are testing the same class. 

Finally, as you can see in our individual tests we are testing the behavior of various methods of the `SettingsShortcutService`. Let me explain the second test in a bit more detail:

```dart
final commandShortcuts = service.getShortcutsFromJson(shortcutsJson);
```

This line calls a method `getShortcutsFromJson(shortcutsJson)` of the service class that we are testing and saves the response in a variable. Note that we are passing a JSON map to this method. We expect a `List<CommandShortcutEvent>`, which is a list of Shortcuts in return for this method.

```dart

final cursorUpShortcut = commandShortcuts.firstWhere((el) => 
    el.key == "move the cursor upward");
final cursorDownShortcut = commandShortcuts.firstWhere((el) => 
    el.key == "move the cursor downward");
```

Here we are tapping into an individual object in the list of commandShortcuts.

```dart
expect(commandShortcuts.length,3);
expect(cursorUpShortcut.command, "alt+arrow up");
expect(cursorDownShortcut.command, "alt+arrow down");
```

Finally, we verify that the actual result matches the expected result. In this particular case, we are checking if the List<ShortcutEvents> generated from a JSON Map we passed to our method under test, is a list of size three elements and we check if the key bindings are appropriate as well. 

Don't worry if you don't understand the actual test, I want you to focus on how it's done, rather than what is done since what is done depends on a broader context about what is being tested. 

## Bloc Tests

AppFlowy uses Bloc for state management. This helps in a clear separation of the Business Logic from the Presentation Logic. Another reason why Bloc library is so popular is because:

> Bloc was designed to be extremely easy to test.

Considering the scope of our article, we shall review the existing Bloc Tests we have in AppFlowy, but in case you want more in-depth knowledge on Bloc Testing, visit their [official docs](https://bloclibrary.dev/#/testing)

While writing bloc tests keep this in mind:

 * are used to verify the behavior of your blocs and cubits.
 * verify whether the UI updates according to state change.
 * mock external dependencies when possible

To write your bloc tests, you will need to create a new file within the directory: 

```
frontend\appflowy_flutter\test\bloc_tests. 
```

Let's take a look at some examples: 

Our first example tests a cubit called `ShortcutsCubit`. This cubit is responsible for handling the business logic behind Customize Shortcuts feature. It depends on a `SettingsShortcutService` instance to contact with the File System for saving and fetching customized shortcuts. 

You can find the cubit here: 
```
appflowy_flutter\lib\workspace\application\settings\shortcuts\settings_shortcuts_cubit.dart
```

Our cubit is depending on `SettingsShortcutService`. In Bloc Tests we want to only focus on the logic of the cubit or bloc, we assume that all the dependencies our cubit or bloc has are working correctly. This can be achieved in Testing by Mocking those dependencies. 

Mocking is a process where we return dummy data and imitate the behavior of a dependency in order to make our Blocs or Cubits work and test them in isolation. 

This is important, because imagine if you don't mock external dependency and your tests fail then it becomes hard to determine whether the point of failure was the external dependencies or the module that you actually wanted to test.

So let's take a look at how we mock external dependencies and set up the required environment for our tests:

```dart

// appflowy_flutter\test\bloc_test\shortcuts_test\shortcuts_cubit_test.dart
import 'package:mocktail/mocktail.dart';

// mocking our dependencies using Mock class by mocktail package
class MockSettingsShortcutService extends Mock implements SettingsShortcutService {}

void main() {
   group("ShortcutsCubit", () {
    late SettingsShortcutService service;
    late ShortcutsCubit shortcutsCubit;

    setUp(() async {
      service = MockSettingsShortcutService();
      when(
        () => service.saveAllShortcuts(any()),
      ).thenAnswer((_) async => true);
      when(
        () => service.getCustomizeShortcuts(),
      ).thenAnswer((_) async => []);
      when(
        () => service.updateCommandShortcuts(any(), any()),
      ).thenAnswer((_) async => Void);

      shortcutsCubit = ShortcutsCubit(service);
    });
    ...
});
```

For mocking the `SettingsShortcutService` we use the Mock class from the mocktail package. Typically the working of our cubit or bloc depends on the methods and data provided from these dependencies. 

Since we are mocking the dependencies, we also have to mock their methods that our cubit will call.
In our case, inside our `setUp` method we first initialize the `SettingsShortcutService` variable with a mock instance. Then we mock its method calls by the `when()` method from the mocktail package.

```dart
when(
 () => service.saveAllShortcuts(any()),
).thenAnswer((_) async => true);
```

The above statement means, when `service.saveAllShortcuts(any())` method is called, answer or return with a true boolean value. The `any()` here means that answer true for **any** input.

Finally in the setUp method we create a new instance of the cubit and pass it the shortcut service.

Now that we have completed our setUp, it is time to jump in the actual test. Let us take a look at the test group fetchShortcuts:

```dart
// appflowy_flutter\test\bloc_test\shortcuts_test\shortcuts_cubit_test.dart
...
group('fetchShortcuts', () {
      blocTest<ShortcutsCubit, ShortcutsState>(
        'calls getCustomizeShortcuts() once',
        build: () => shortcutsCubit,
        act: (cubit) => cubit.fetchShortcuts(),
        verify: (_) {
          verify(() => service.getCustomizeShortcuts()).called(1);
        },
      );

      blocTest<ShortcutsCubit, ShortcutsState>(
        'emits [updating, failure] when getCustomizeShortcuts() throws',
        setUp: () {
          when(
            () => service.getCustomizeShortcuts(),
          ).thenThrow(Exception('oops'));
        },
        build: () => shortcutsCubit,
        act: (cubit) => cubit.fetchShortcuts(),
        expect: () => <dynamic>[
          const ShortcutsState(status: ShortcutsStatus.updating),
          isA<ShortcutsState>()
              .having((w) => w.status, 'status', ShortcutsStatus.failure)
        ],
      );
      ...
    });
  ...
}
```

Note how we don't use the `test()` method to test our cubit, instead we use the `blocTest()` method. 

In our first bloc test, at the top, we have the test description which states that we want to verify whether a certain `getCustomizeShortcuts()` method was called or not? To do so we pass a callback to the build method which is responsible for building the cubit under test.

```dart
build: () => shortcutsCubit,
```

then we pass an action to be taken with that cubit in the act parameter. In this case we want to call the `fetchShortcuts()` method of our cubit under test. When writing tests for Blocs instead of cubit, the act parameter will expect an Event class instance for that Bloc instead of a method call.

```dart
act: (cubit) => cubit.fetchShortcuts(),
```

Finally we verify if the service's (the dependency that we mocked) `getCustomizeShortcuts()` method was called once or not. 

```dart
verify: (_) {
 verify(() => service.getCustomizeShortcuts()).called(1);
},
```

That's it for the first test. The test verifies whether a certain method inside provided to our cubit by an external dependency was called or not. 

Since we have mocked the `getCustomizeShortcuts()` method, we don't actually execute the original method. Thus if a failure were to occur in this test, we will know that it was our cubit to be held responsible for the failing test and the method is completely innocent.

That is great for verifying whether some method was called or not, but how can I assert expectations for my cubit or blocs. Let us take a look at the second test in this group. 

```dart
...
blocTest<ShortcutsCubit, ShortcutsState>(
        'emits [updating, failure] when getCustomizeShortcuts() throws',
        setUp: () {
          when(
            () => service.getCustomizeShortcuts(),
          ).thenThrow(Exception('oops'));
        },
        build: () => shortcutsCubit,
        act: (cubit) => cubit.fetchShortcuts(),
        expect: () => <dynamic>[
          const ShortcutsState(status: ShortcutsStatus.updating),
          isA<ShortcutsState>()
              .having((w) => w.status, 'status', ShortcutsStatus.failure)
        ],
);
...
```

In this test we expect our cubit to emit the Failure state due to some exception in the service that our cubit depends upon.

A few new things here, first of all there is a `setUp` parameter inside our blocTest. Initially when we mocked this method of `getCustomizeShortcuts()` inside our top level `setUp()` method, we were returning an empty list, which was a output representing a success operation. 

But in this test we are overriding that mocked implementation to represent a failure state by throwing an exception. To throw an exception we call the `thenThrow()` method with an Exception instance as a parameter.

Therefore we can test if our Cubit is able to handle this failure and produce appropriate result for the Cubit Consumer or Builder. 

```dart
setUp: () {
        when(
            () => service.getCustomizeShortcuts(),
        ).thenThrow(Exception('oops'));
},
```

After overriding the mock implementation, we build our cubit and call the `fetchShortcuts()` method. Finally to end the test we expect an array or list of `ShortcutsState`. The last value in this list is an instance of `ShortcutsState` with the status set to Failure.

```dart
expect: () => <dynamic>[
        const ShortcutsState(status: ShortcutsStatus.updating),
        isA<ShortcutsState>()
            .having((w) => w.status, 'status', ShortcutsStatus.failure)
        ],
```

Similarly we test all the methods and events in our cubits or blocs, to verify their proper working. I request the reader to try out Bloc testing for themselves using the official docs mentioned above to get more clear understanding for the same.