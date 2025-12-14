# Localization

Localization (_l10n_) is done through the Flutter [intl package](https://pub.dev/packages/intl).
This page shows how to create localized entries, parametrize them and access them in code after 
generating localization files.
Also, some project-individual localization features are shown.

## Create and Access Localizations

Localization ARB files can be found in `lib/l10n/` folder.
Localized entries are created as key-value-pairs, looking like this:
```yaml
  "authLoginTitle": "Login",
```

After localizations have been edited, corresponding Dart files have to be re-generated using the
following command:
```sh
  dart run intl_utils:generate
```

> There are also tools automatically generating intl files while programming.
> A good example is the 
> [Android Studio intl plugin](https://plugins.jetbrains.com/plugin/13666-flutter-intl), which can 
> significantly speed up localization process.
{style="note"}

A localized entry is loaded using `S.of(context)`, suffixing the name of the key like that:
```dart
  Text(S.of(context).authLoginTitle)
```

One can add parameters to a localized entry by writing a parameter in curly braces.
In such cases, a descriptive localization entry should be created, emphasizing the parameter's
meaning:
```yaml
  "authLoginFailed": "Login failed. (Status {{status}})",
  "@authLoginFailed": {
    "description": "status: HTTP status code of the login"
  }
```

Parameters are passed to the key when accessing the localized item:
```dart
  Text(S.of(context).authLoginFailed(e.toString()))
```

> For other features of intl packages, please refer to the 
> [package information](https://pub.dev/packages/intl).

## LocaleController

The `LocaleController` is the central class for handling all logical operations related to 
localization within the app.
The controller can be accessed anytime using the widget tree over the current `BuildContext` 
like so:
```dart
  context.read<LocaleController>()
```

There are some interesting data you may access:
- `locale`: Returns the current `Locale` object, reflecting the selected language.
- `currentLocaleIsGerman`: Utility to quickly access whether the user currently has a German 
  language selection entry.
- `translations`: Loads current localized entries without the need of `BuildContext`. This can 
  be useful when accessing localizations in static repositories.

## TranslatedText

`TranslatedText` is a utility model reflecting API-sent entries which are available in multiple 
languages. Each `TranslatedText` therefore stores any amount of `Translation`-entries, each 
reflecting a specific translation `text` with its corresponding `languageCode`.

`TranslatedText` offers the following utilities:
- `translations`: The full list of all `Translation` items.
- `getTextForLanguageCode`: Returns the relevant translation for a given locale code. If there 
  is no fitting translation, the English localization is returned. If there is no translation at 
  all, returns `null`.
- `getTextForControllerLanguage`: Same as `getTextForLanguageCode`, but considers the locale 
  currently set in the `LocaleController`.
- `getTextForCurrentLanguage`: Same as `getTextForControllerLanguage`, but loads the 
  `LocaleController` through the passed `BuildContext`.
- `german`: Same as `getTextForLanguageCode`, but tries to load the German entry.
- `english`: Same as `getTextForLanguageCode`, but tries to load the English entry.

There also is a service model implementation for the HSMWmobil app API called 
`AppApiTranslatedText` with a fitting `fromJson` constructor.