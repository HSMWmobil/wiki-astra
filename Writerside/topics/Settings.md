# Settings

## Why is there a settings system, and why is it important?

The app includes user preferences that can be configured in various places.
To ensure these preferences are reliably saved and centrally managed,
a broad settings system was developed that allows settings to be saved and adjusted both locally and remotely.

Without this system, each feature would have to save and load its own preferences,
which would lead to inconsistencies in various places.

At the same time, the system makes it possible to largely separate the app’s features from the settings.
The features only need to know which settings they offer, not where or how they are stored.

## What and why are `UserSettings`?

`UserSettings` describe the user's personal preferences, which can be configured and saved in the app.
For instance, dark mode, language, etc. These settings are handled centrally within the app.

Using just one `UserSettings` model therefor has the advantage of standardizing
how the various settings are read.

## The `UserSettings` model

The preferences are organized within a single flat map,
which the app uses to read user preferences and apply updates.
Each entry is assigned a unique string and uses a prefix to categorize the setting.
The value of each entry can be of any JSON-compatible type.

```json
{
  "timetable.showWeekends": true,
  "timetable.showWeekNumbers": false,
  "canteen.showPrices": true,
  "canteen.showOpeningHours": false
}
```

## `SettingsController`

### Responsibilities

The `SettingsController` provides a central entry point for accessing and changing user settings.
This keeps the rest of the app independent of the underlying storage and synchronization logic.
It is also responsible for notifying the UI of changes and maintains the currently active preferences.
The controller does not know whether the settings are stored locally or remotely.
Instead, it receives the `SettingsRepository` and handles loading and saving accordingly.

### Current Settings

The controller provides the currently active settings via `currentSettings`.
This object can be used by other parts of the app to read values without having
to access storage or the remote API.

```dart
final settings = context.watch<SettingsController>();
final isDarkMode = settings.currentSettings.getBool(
  UserSettingsKeys.designDarkMode,
);
```

### Initialization with `initialize()`

The `initialize()` method allows the controller to load the initial settings from the repository.
If no user preferences are available yet, the default settings are created using `UserSettingsDefaults`.

Once the initial state has been set, the controller listens for updates from the repository.
Each update replaces the previous `UserSettings` instance and notifies the associated listeners.

### Updating Settings with `patchSettings()`

Settings are changed using the patchSettings() method.
The caller passes only the value that has to be changed. 
The controller then combines this value with the current settings and the default settings,
and passes the entire settings map to the repository.

The controller updates its own state after the repository has been successfully updated.
After that, the corresponding widgets are rebuilt via `notifyListeners()`.

### Update Order

If multiple changes are triggered within a short period of time, 
the controller uses an internal queue to process the requests sequentially.
This prevents updates from accidentally getting overwritten by an earlier asynchronic write.

### Clearing App Data and dispose()

The clearAppData() method deletes all stored data within getStorage. 
In addition, the controller's current user settings are reset to their default values, 
and the relevant listeners are notified.
The SettingsScreen then restarts the application so that all controllers 
are reinitialized with cleared local data.

To delete the remote data stored in Astra, 
you must instead call the `SettingsRepository.clear()` method.

When the controller is initialized, it subscribes to a settings stream.
When the controller is no longer needed, `dispose()` ensures that the subscription 
is canceled and the associated resources get released.
It then calls `super.dispose()` to complete the disposal process of
`ChangeNotifier`.

## Keys and Default Values

### `UserSettingsKeys`

`UserSettingsKeys` contains the canonical keys that can be used to access the corresponding preferences.
Each one is defined as a constant and follows a prefix-based naming convention.
A central key definitions prevents typos and inconsistencies when accessing the settings map.

```dart
static const String timetableDisplayPeriod =
    'timetable.displayPeriod';
 ```
The constant name ist used in the source code, while its string value is used 
as the actual key in the settings map.

```
Source code: UserSettingsKeys.timetableDisplayPeriod: 'oneYear',
Actual key-value pair in the settings map: 'timetable.displayPeriod': 'oneYear',
```

### `UserSettingsDefaults`

To ensure that the app is working conveniently even if no user preferences have been saved yet,
`UserSettingsDefaults` contains the initial values for all configurable preferences. 
These values are stored in a shared map and use the keys from `UserSettingsKeys`.

```dart
static final Map<String, dynamic> values = {
  UserSettingsKeys.timetableDisplayPeriod: 'oneYear',
  UserSettingsKeys.timetableApplyEnrollment: true,
};
```

### Adding and Changing keys/ settings

When a new setting is created, its key must be added under `UserSettingsKeys`.
This key should not be changed during development.
If you rename an existing key, you may need to perform a migration 
for settings that have already been saved under the old key.

## Storage and Synchronization

The app uses different repositories for local and remote storage.
The `SyncingUserSettingsRepository` coordinates both.

### Local Storage

The `LocalSettingsRepository` stores settings using `getStorage`.
When a user is authenticated, their preferences are stored under their username.
This allows multiple users on the same device to have different local settings.

Local storage also allows preferences to be changed and saved 
even if the network connection is disrupted.

### Remote Storage through Astra

Remote storage allows user preferences to be restored on a different device or after reinstalling the app.
The `AstraUserSettingsRepository` therefor communicates with the Astra API endpoint `lists/remote_settings`.
Remote settings are sent to Astra as one complete map:

```json
{
  "is_public": false,
  "data": {
    "design.darkMode": true,
    "timetable.calendarSync": false,
    "...": "..."
  }
}
```
The remote repository reads the preferences from the `data` object 
and writes changes using the same structure.

### `SyncingUserSettingsRepository`

The `SyncingUserSettingsRepository` combines the local and remote repositories.
Local preferences are loaded first, so the app remains usable without a network connection.
When a user logs in or the user changes, 
the repository activates the local settings and loads the corresponding remote settings.

When a preference is changed, the local value is saved first. 
The settings map is then sent to Astra (provided the user is authenticated). 
If this synchronization fails, 
the local change is preserved and marked for a later synchronization attempt.

If multiple devices update the settings for the same account, 
the last change will overwrite the previous one.


## Using Settings in the App

The `SettingsController` is registered globally in the `bootLoader`.
This allows other widgets and controllers to access the user settings via the provider.
The settings should always be accessed through the controller.

