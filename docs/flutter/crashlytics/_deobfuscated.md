{# This content gets published to the following location:                                   #}
{#   https://firebase.google.com/docs/crashlytics/get-deobfuscated-reports?platform=flutter #}

By default, {{firebase_crashlytics}} automatically instruments your Flutter
project to upload the necessary symbol files that ensure crash reports are
deobfuscated and human readable.

Unfortunately, there are cases that can result in the project not being fully
configured. This guide outlines what the automation does and provides first
steps to debug your project setup.

## Apple platforms {: #apple}

### Check your configuration for uploading dSYM files {: #apple-upload-dsym-script-configuration}

Adding the {{crashlytics}} Flutter plugin and running the
`flutterfire configure` command will attempt to add a run script to your
project’s Xcode workspace that finds and uploads the necessary dSYM symbol files
to {{crashlytics}}. Without these files, you’ll see a "Missing dSYM" alert in
the {{crashlytics}} dashboard and exceptions will be held by the backend until
the missing files are uploaded.

If you have this issue, first make sure that you have the run script installed:

1.  Locate and open the Xcode workspace file in your project’s iOS directory
    (<code><var>FLUTTER_PROJECT_NAME</var>/ios/Runner.xcworkspace</code>).

1.  Identify whether a run script titled
    `[firebase_crashlytics] Crashlytics Upload Symbols` has been added to the
    Runner target’s Build Phases.

    See the applicable section below for whether the
    [run script does _not_ exist](#run-script-does-not-exist) or the
    [run script exists](#run-script-exists).

{{ '<section class="expandable" id="run-script-does-not-exist">' }}
<h4 class="showalways" id="run-script-does-not-exist">Run script for auto-upload of dSYMs does _not_ exist</h4>

If this run script does _not_ exist, you can add it manually:

1.  Locate the Firebase App ID for your Apple app. Here are two different places
    where you can find this ID:

    * In the {{name_appmanager}}, go to your
      <nobr><span class="material-icons">settings</span> > _Project settings_</nobr>.
      Scroll down to the _Your apps_ card, then click on your
      Firebase Apple App to view the app's information, including its _App ID_.

    * In your Flutter project's top-level directory, find your
      `firebase_options.dart` file. The Firebase App ID for your Apple app is
      labeled as `GOOGLE_APP_ID`.

1.  Click <span class="material-icons">add</span> >
    **New Run Script Phase**.

    Make sure this new _Run Script_ phase is your project's last build
    phase. Otherwise, {{crashlytics}} can't properly process dSYMs.

1.  Expand the new _Run Script_ section.

    Note: For the remaining substeps, copy-and-paste the paths exactly as
    specified, and Xcode will resolve them. However, if you have issues with
    Xcode resolving these paths or a unique project structure, you can
    manually specify the paths instead.

1.  In the script field (located under the _Shell_ label), add the
    following run scripts.

    These scripts process your dSYM files and upload the files to
    {{crashlytics}}.

    <pre class="devsite-click-to-copy">"$PODS_ROOT/FirebaseCrashlytics/upload-symbols" --build-phase --validate -ai "<var>FIREBASE_APP_ID</var>" -- "$DWARF_DSYM_FOLDER_PATH/$DWARF_DSYM_FILE_NAME"</pre>

    <pre class="devsite-click-to-copy">"$PODS_ROOT/FirebaseCrashlytics/upload-symbols" --build-phase -ai "<var>FIREBASE_APP_ID</var>" -- "$DWARF_DSYM_FOLDER_PATH/$DWARF_DSYM_FILE_NAME"</pre>

    * <var>FIREBASE_APP_ID</var>: Your Firebase Apple App ID (not your
      Apple bundle ID)<br>
      Example Firebase Apple App ID: `1:1234567890:ios:321abc456def7890`

    {{ '<section class="expandable">' }}
    <p class="showalways">Need to find your Firebase App ID?</p>

    > Here are two ways to find your Firebase App ID:
    >
    > * In your `GoogleService-Info.plist` file, your App ID is the
    >   `GOOGLE_APP_ID` value; or
    >
    > * In the {{name_appmanager}}, go to your
    >   [_Project settings_](https://console.firebase.google.com/project/_/settings/general/){: .external}.
    >   Scroll down to the _Your apps_ card, then click on the desired Firebase
    >   App to find its App ID.

    {{ '</section>' }}

1.  In the _Input Files_ section, add the paths for the locations of the
    following files:

    <pre class="devsite-click-to-copy">"${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}"</pre>

    <pre class="devsite-click-to-copy">"${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}/Contents/Resources/DWARF/${PRODUCT_NAME}"</pre>

    <pre class="devsite-click-to-copy">"${DWARF_DSYM_FOLDER_PATH}/${DWARF_DSYM_FILE_NAME}/Contents/Info.plist"</pre>

    <pre class="devsite-click-to-copy">"$(TARGET_BUILD_DIR)/$(UNLOCALIZED_RESOURCES_FOLDER_PATH)/GoogleService-Info.plist"</pre>

    <pre class="devsite-click-to-copy">"$(TARGET_BUILD_DIR)/$(EXECUTABLE_PATH)"</pre>

    <section class="expandable">
      <p class="showalways"><b>Understand why the locations of these files are
        needed</b>
      </p>

      <p>Xcode looks in the specified locations for these input files to ensure
        that the build files are available for the run script. Also, if
        <i>User Script Sandboxing</i> is enabled, Xcode only allows the run
        script to access files specified in the <i>Input Files</i>.
      </p>

      <ul>
        <li>Providing the location of your project's dSYM files enables
          {{crashlytics}} to process dSYMs.
        </li>
        <li>Providing the location of your app's built
          <code>GoogleService-Info.plist</code>
          file enables {{crashlytics}} to associate the dSYMs with your
          Firebase app.
        </li>
        <li>Providing the location of your app's executable allows the run
          script to prevent duplicate uploads of the same dSYM. Note that app
          binaries are <em>not uploaded</em>.
        </li>
      </ul>

    </section>

{{ '</section>' }}

{{ '<section class="expandable" id="run-script-exists">' }}
<h4 class="showalways" id="run-script-exists">Run script for auto-upload of dSYMs exists</h4>

If the run script does exist, refer to the
[Apple-specific guide for troubleshooting dSYM issues](/docs/crashlytics/get-deobfuscated-reports?platform=ios).
You’ll need to take the following additional steps if you choose to upload your
dSYM files via the described process:

1.  Locate the Firebase App ID for your Apple app. Here are two different places
    where you can find this ID:

    * In the {{name_appmanager}}, go to your
      <nobr><span class="material-icons">settings</span> > _Project settings_</nobr>.
      Scroll down to the _Your apps_ card, then click on your
      Firebase Apple App to view the app's information, including its _App ID_.

    * In your Flutter project's top-level directory, find your
      `firebase_options.dart` file. The Firebase App ID for your Apple app is
      labeled as `GOOGLE_APP_ID`.

1.  When running the `upload-symbols` script, use
    <code><nobr>-ai <var>FIREBASE_APPLE_APP_ID</var></nobr></code> instead of
    <nobr><code>-gsp /path/to/GoogleService-Info.plist</code></nobr>.

{{ '</section>' }}

### Check your version configuration for Flutter and {{crashlytics}} _(if using the `--split-debug-info` flag)_ {: #apple-split-debug-info}

If your Flutter project uses the `--split-debug-info` flag (and, optionally,
also the `--obfuscate` flag), additional steps are required to show readable
stack traces for your app.

Make sure that your project is using the recommended version configuration
(Flutter 3.12.0+ and {{crashlytics}} Flutter plugin 3.3.4+) so that your project
can automatically generate and upload Flutter symbols (dSYM files) to
{{crashlytics}}.


## Android {: #android}

### Check your dependency configuration {: #android-dependency-configuration}

The `flutterfire configure` command attempts to add necessary dependencies to
your project’s Gradle build files. Without these dependencies, crash reports in
the {{name_appmanager}} may end up obfuscated if obfuscation is turned on.

Make sure the following lines are present in the project-level `build.gradle`
and in the app-level `build.gradle`:

* In the **project-level** build file (`android/build.gradle`), check for the
  following line:

  <pre class="prettyprint">
  dependencies {
    // ... other dependencies

    classpath 'com.google.gms:google-services:4.3.5'
    <strong>classpath 'com.google.firebase:firebase-crashlytics-gradle:2.7.1'</strong>
  }
  </pre>

* In the **app-level** build file (`android/app/build.gradle`), check for the
  following line:

  <pre class="prettyprint">
  // ... other imports

  android {
    // ... your android config
  }

  dependencies {
    // ... your dependencies
  }

  // This section must appear at the bottom of the file
  apply plugin: 'com.google.gms.google-services'
  <strong>apply plugin: 'com.google.firebase.crashlytics'</strong>
  </pre>

### Check that you're using the {{cli}} to upload Flutter symbols _(if using the `--split-debug-info` flag)_ {: #android-split-debug-info}

If your Flutter project uses the `--split-debug-info` flag (and, optionally,
also the `--obfuscate` flag), additional steps are required to show readable
stack traces for your app.

Use the [{{firebase_cli}}](/docs/cli) (v.11.9.0+) to upload Flutter debug
symbols. You need to upload the debug symbols _before_ reporting a crash from an
obfuscated code build.

From the root directory of your Flutter project, run the following command:

<pre class="devsite-terminal" data-terminal-prefix="your-flutter-proj$ ">firebase crashlytics:symbols:upload --app=<var class="readonly">FIREBASE_APP_ID</var> <var class="readonly">PATH/TO</var>/symbols</pre>

* <var>FIREBASE_APP_ID</var>: Your Firebase Android App ID (not your
  package name)<br>
  Example Firebase Android App ID: `1:567383003300:android:17104a2ced0c9b9b`

    {{ '<section class="expandable">' }}
    <p class="showalways">Need to find your Firebase App ID?</p>

    > Here are two ways to find your Firebase App ID:
    >
    > * In your `google-services.json` file, your App ID is the
    >   `mobilesdk_app_id` value; or
    >
    > * In the {{name_appmanager}}, go to your
    >   [_Project settings_](https://console.firebase.google.com/project/_/settings/general/){: .external}.
    >   Scroll down to the _Your apps_ card, then click on the desired Firebase
    >   App to find its App ID.

    {{ '</section>' }}

* <code><var>PATH/TO</var>/symbols</code>: The same directory that you
  pass to the `--split-debug-info` flag when building the application

If problems persist, refer to the
[Android-specific guide for troubleshooting obfuscated reports](/docs/crashlytics/get-deobfuscated-reports?platform=android).
