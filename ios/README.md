# Mutqin iOS wrapper

This folder contains a small native iOS wrapper for the Surah Al-Hajj memorization app.

The app is a SwiftUI shell with a `WKWebView` that loads the existing `surah-al-hajj-memorization.html` file from the app bundle. Xcode runs a build phase named **Sync Web App** before each build, copying the latest root HTML file into `Mutqin/Resources/`.

## Run on your iPhone with a free Apple ID

1. Install Xcode from the Mac App Store.
2. Open `ios/Mutqin.xcodeproj`.
3. In Xcode, open **Settings > Accounts** and add your Apple ID.
4. Select the **Mutqin** project, then the **Mutqin** target.
5. Open **Signing & Capabilities**.
6. Choose your personal team.
7. If Xcode says the bundle identifier is unavailable, change `com.selim6167.mutqin` to something unique, for example `com.yourname.mutqin`.
8. Connect your iPhone by USB.
9. On the iPhone, enable **Developer Mode** if iOS asks for it.
10. Select your iPhone as the run destination and press **Run**.

With a free Apple ID, the app is for personal installation. You may need to reconnect to Xcode and reinstall periodically when the free signing profile expires.

Progress saved inside this native app is separate from Safari/GitHub Pages because it uses the native app web view's own local storage.
