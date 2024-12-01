# Adding Custom Vendor Packages to Android Source Code: A Complete Guide

As Android developers, we often need to integrate custom applications and packages into the Android source code. This guide will walk you through the process of adding a custom vendor and its associated packages to your Android source tree.

## Directory Structure Setup

First, create the following directory structure in your Android source code:

```
vendor/
└── oem/
    └── packages/
        └── applications/
            ├── Android.bp
            ├── packages.mk
            ├── permissions/
            │   └── com.sample.app.xml
            └── prebuilts/
                ├── SampleApp.apk
                ├── SampleAppWithPermission.apk
                └── SampleLauncher.apk
```

## Configuration Files

### 1. Android.bp

The `Android.bp` file defines how your applications should be built and installed. Here's a breakdown of different types of app configurations:

```blueprint
// Basic Launcher Application
android_app_import {
    name: "SampleLauncher",
    privileged: true,
    apk: "prebuilts/SampleLauncher.apk",
    overrides: [
        "Launcher2",
        "Launcher3",
        "Launcher3QuickStep",
        "CarLauncher",
        "SnapdragonLauncher",
    ],
    presigned: true,
}

// Simple System Extension Application
android_app_import {
    name: "SampleApp",
    system_ext_specific: true,
    apk: "prebuilts/SampleApp.apk",
    optional_uses_libs: [
        "androidx.window.extensions",
        "androidx.window.sidecar",
    ],
    presigned: true,
}

// Privileged Application with Permissions
android_app_import {
    name: "SampleAppWithPermission",
    system_ext_specific: true,
    privileged: true,
    optional_uses_libs: [
        "androidx.window.extensions",
        "androidx.window.sidecar",
    ],
    apk: "prebuilts/SampleAppWithPermission.apk",
    required: ["privapp_whitelist_com.sample.app.xml"],
}

// Permission Configuration
prebuilt_etc {
    name: "privapp_whitelist_com.sample.app.xml",
    system_ext_specific: true,
    src: "permissions/com.sample.app.xml",
    sub_dir: "permissions",
    filename_from_src: true,
}
```

### 2. packages.mk

Create a `packages.mk` file to define which packages should be included in the build:

```makefile
# Packages
PRODUCT_PACKAGES += \
    SampleLauncher \
    SampleApp \
    SampleAppWithPermission
```

### 3. Permission XML

For applications requiring privileged permissions, create a permission XML file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<permissions>
    <privapp-permissions package="com.sample.app">
        <permission name="android.permission.CAPTURE_AUDIO_OUTPUT"/>
    </privapp-permissions>
</permissions>
```

## Integration with Build System

To integrate your custom vendor packages into the Android build system, you need to include the packages.mk file in your device's product configuration. There are several ways to do this depending on your setup:

1. For standard handheld devices, add this line to `build/make/target/product/handheld_product.mk`:
```makefile
include vendor/oem/packages/applications/packages.mk
```

2. For Qualcomm-based devices, you might need to add it to:
   - `device/qcom/qssi/qssi.mk`
   - `device/qcom/qssi/qssi_au.mk`
   - Or your specific device's product makefile

## Important Notes

1. **APK Signing**: Ensure your APKs are properly signed before adding them to the prebuilts directory. The `presigned: true` flag indicates that the APK is already signed.

2. **Permissions**: For privileged applications, make sure to:
   - Set `privileged: true` in the Android.bp configuration
   - Create appropriate permission XML files
   - Reference the permission XML in the `required` field

3. **System Extensions**: Use `system_ext_specific: true` for apps that should be installed in the system_ext partition.

4. **Override Existing Apps**: Use the `overrides` array to replace existing system applications, as shown in the SampleLauncher configuration.

## Conclusion

By following this guide, you can properly integrate custom vendor packages into your Android source code. Remember to test thoroughly and ensure all permissions are properly configured before building your system image.

Remember that the exact integration path might vary depending on your specific device tree and BSP (Board Support Package) configuration. Always consult your device's documentation for the most appropriate location to include custom packages.
