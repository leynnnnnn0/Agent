# Portable Codex Android rules and implementation reference

This single file is the complete handoff. The next agent may have ONLY this file, with no original project, screenshots, chat history, helpers, or drawables. All required modal, toast, badge, model, session, and database templates are embedded below. Recreate them from these blocks when missing; reuse them when already present. Do not say “use the existing helper” without first checking that it exists.

The examples use `com.example.app`. Replace that package, its `R` imports, and source directories with the destination application's package. Merge resource entries into existing files rather than overwriting unrelated project resources. Create only the components needed by the requested feature; this reference is not an instruction to generate every file immediately.

## 1. Hard rules

- Java application code and XML layouts only, unless the user explicitly requests otherwise.
- OFFLINE ONLY: the app must be developed, built, and run using the Android Studio installation, SDK, build tools, assets, and dependencies already available on that computer. Internet access must not be required.
- Android SDK APIs and locally available libraries are allowed. AndroidX RecyclerView, AppCompat, ConstraintLayout, Material Components, ChipGroup, and other components are permitted when their complete dependencies are already present locally and they work offline.
- The restriction is NOT “SDK-only” and NOT a ban based on who published a library. A third-party library is acceptable only if it is already locally available, relevant to the requested feature, and needs neither a download nor an online service. Prefer existing project components over adding complexity.
- Never download libraries, Gradle plugins, SDK packages, fonts, icons, images, or other assets. Do not change dependency versions to ones unavailable locally or depend on remote repositories resolving new artifacts.
- Never use external web APIs, HTTP backends, cloud databases, online authentication, remote image URLs, or services that need a network connection. Local Android framework APIs and local SQLite calls are allowed; “no APIs” refers to external/network services, not the Android SDK itself.
- Do not add the INTERNET permission or network client code for these offline projects. Store and process application data locally through the required DBHelper and other requested local functionality.
- Keep the destination project's existing offline-compatible AndroidX/Material setup. Do not rewrite it into SDK-only code merely because the earlier version of this guide called for “native only.” The platform-only templates below are complete fallback implementations, not a prohibition against locally available widgets.
- Keep implementations short, direct, and readable. Do not add abstraction layers, repositories, dependency injection, builders, configuration objects, listeners, generic frameworks, or extra files without an actual need.
- Do not minify code or combine unrelated statements just to claim fewer lines.
- NEVER add Java comments, Javadocs, XML comments, TODOs, or commented-out code. Explanations belong in prose outside the source.
- Adapter constructors take ONLY `Context context` and the model list. No third argument, callback, listener, consumer, or options object. Action logic stays inside the adapter.
- NEVER use SQLite foreign-key constraints or nested model relationships. Store plain integer fields such as `userId`, `studentId`, and `subjectId`; section 22 shows the exact rule.
- All state-changing transactions require a confirmation/danger modal BEFORE execution and success/error toast AFTER the actual result. Cancel changes nothing.
- Handle real errors, close cursors/streams, and reset recycled view state. Do not omit essential correctness to reduce code length, and do not add speculative checks unrelated to the feature.
- Reuse resources. Remove files and entries made unused by a change, while keeping shared resources still in use.
- Do not reintroduce image saving, a dedicated success page, or a custom loading button unless the user asks for them.
- For an application built from a machine problem, create the tables required by that problem and seed every created table with useful local sample records. Also seed supporting tables and entities needed to make the requested workflows and screens usable, even when their example records are not supplied by the problem. Do not create unrelated features, a login flow, or extra screens without a real use in the application.

## 2. Architecture and naming

| Package/resource | Responsibility |
| --- | --- |
| Base package or role/feature package such as `admin` | Activities, screen setup, navigation, refreshing records |
| `models` | Plain public-field model classes and `StaticUserModel` |
| `adapters` | RecyclerView adapters, ViewHolders, binding, row action logic |
| `helpers` | `DBHelper`, `AppModal`, `AppToast`, genuinely shared behavior |
| `res/layout` | Activity layouts and reusable component layouts |
| `res/drawable` | XML backgrounds and vector icons |
| `res/values` | Shared strings, colors, and styles |

Keep SQL in DBHelper and fields in models. Do not put a second database implementation inside an Activity or adapter. Use role packages only when there is a real screen to organize. Do not invent MVVM/MVC layers for this small Java/XML reference.

## 3. XML and Java formatting

Use four-space indentation. Put the opening element name on its own line and EVERY XML attribute on a separate line. Do not put multiple attributes beside one another. This applies to layouts, drawables, menus, values files, and the manifest; the XML declaration can remain on one line.

```xml
<TextView
    android:id="@+id/user_phone"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:drawableStart="@drawable/ic_user_phone"
    android:text="@string/sample_user_phone" />
```

Keep source free of comments. Use clear variable names, plain public model fields, and no unnecessary getters/setters. Follow the destination project's normal Java formatting. Do not add optional decorative `importantForAccessibility`, `contentDescription`, accessibility-heading, or live-region attributes by default; the user explicitly prefers minimal hackathon XML. Add them only when requested or essential to a requested feature.

For preview-only placeholders use the built-in Android Studio `tools` namespace. It is design-time metadata, not a dependency. Use `tools:text`, `tools:background`, and `tools:scaleX`/`tools:scaleY` so preview content is visible without replacing runtime data. Every modal/toast layout below includes these.

## 4. Exact component appearance and behavior

These specifications and the complete sample code/resources below are the fixed design. Use them exactly for the modal and toast; do not restyle their shapes, gradient colors, icon rings, animation, placement, type sizes, or buttons. If the project already contains the approved AppModal/AppToast samples, reuse those existing helpers and resources without rewriting them. Do not substitute a stock AlertDialog, Android Toast, Snackbar, or a new theme-derived design. These specifications replace the unavailable screenshots. Do not redesign the components from vague terms such as “modern.” Use the sizes, shapes, and colors below. Font is the system sans-serif. Use dp for dimensions and sp for text.

### Confirmation and danger modal

- Centered floating card, maximum width 420dp, with at least 24dp side space on smaller screens; height wraps its content.
- Rounded card corners 24dp and internal padding 24dp. Dim the rest of the Activity to 55%.
- Confirmation card is a very light diagonal green/white gradient. Danger uses a very light pink/white gradient.
- Centered 96dp circular icon: two pale rings around a colored gradient center, white bold 36sp `?` for confirmation or `!` for danger. Do not add an image library.
- Icon starts at scale 0 and expands to 1 over 1200ms with Android's `OvershootInterpolator`; a small bounce is intentional.
- Centered bold 22sp title, color `#202033`, 12dp below the icon.
- Centered 15sp message, color `#6B6B80`, 10dp below the title and 3dp additional line spacing.
- Buttons are a horizontal equal-width row 28dp below the message. Each is 52dp high, rounded 12dp. Gap is 12dp.
- Cancel is pale/transparent white with a 1dp `#DADDEA` outline and dark `#29293D` text. Confirm is a filled green gradient; danger Confirm is red. Confirm label is `Confirm →`, white 15sp, normal case.
- Both cancel and back/outside dismissal make no change. Confirm dismisses the modal and runs the supplied action once. Prevent repeated confirm taps.
- Do not add a separate success modal. Success is communicated by the top toast.

### Top success/error toast banner

- This is an in-app view overlay, NOT Android's restricted system `Toast`, NOT a dialog, and NOT a notification.
- It spans available Activity width with 12dp side margins. Place it 12dp below the status bar/cutout. It must not dim the screen or stop interaction with the rest of the page.
- Same rounded 24dp gradient card design as the modal: pale green for success, pale pink for error; padding 16dp, elevation 8dp.
- Left icon is 56dp with pale rings and gradient center; white bold 24sp `✓` for success or `!` for error.
- Right text column has 12dp spacing from the icon. Title is bold 16sp `#202033`; message is 14sp `#6B6B80`, with 4dp top spacing. Text wraps naturally.
- Slide/fade in over 300ms from 48dp above its resting position. Stay for 4000ms, then slide/fade out over 200ms.
- Tap the banner to dismiss it. A new banner replaces the previous one instead of stacking. Do not hold Activities in static fields.
- Its text must describe the actual operation result. Do not use success styling for an operation that failed.

### Five reusable badges

- Names are exactly `green_badge`, `red_badge`, `blue_badge`, `yellow_badge`, `purple_badge` for both drawable and style resources.
- Noninteractive `TextView` pill: 100dp corner radius, subtle vertical pastel gradient, 1dp pale outline, minimum height 28dp.
- A solid 8dp circular dot sits 12dp from the left and is vertically centered. Implement the dot inside the background `layer-list`, not as another view.
- Background padding is left 28dp, right 12dp, top/bottom 6dp; this reserves room for the dot. Use `paddingMode="stack"` so the dot is not displaced by nested drawable padding.
- Text is system sans-serif bold 12sp in a darker shade of its color. No forced uppercase, click handling, or fixed width. Label comes from the actual status.

| Badge | Gradient top | Gradient bottom | Outline | Dot | Text |
| --- | --- | --- | --- | --- | --- |
| Green | `#F1FCF8` | `#E6FAF2` | `#D7F6EC` | `#10BE8B` | `#12805C` |
| Red | `#FFF3F5` | `#FFE8ED` | `#FBD5DD` | `#EF4260` | `#BC3048` |
| Blue | `#EFF6FF` | `#E2EEFF` | `#D8E7FF` | `#4385F5` | `#285FB9` |
| Yellow | `#FFFBEB` | `#FFF3CE` | `#FAE9AB` | `#E8B129` | `#946900` |
| Purple | `#F8F2FF` | `#EEDFFF` | `#E8D9FC` | `#9A62DE` | `#7944AD` |

## 5. Complete native component files

The following are complete files, not pseudocode. They require API 24 or newer, Java 8+ language support, and the normal Android SDK. They do not need Material, AppCompat, ConstraintLayout, image libraries, or third-party dependencies. Keep all resource names consistent. When merging into a project, preserve unrelated resource entries.

### `app/src/main/java/com/example/app/helpers/AppModal.java`

```java
package com.example.app.helpers;

import android.app.Dialog;
import android.content.Context;
import android.graphics.Color;
import android.graphics.drawable.ColorDrawable;
import android.view.LayoutInflater;
import android.view.View;
import android.view.Window;
import android.view.WindowManager;
import android.view.animation.OvershootInterpolator;
import android.widget.TextView;
import com.example.app.R;

public final class AppModal {
    public static void confirm(Context context, String title, String message, Runnable onConfirm) {
        show(context, title, message, onConfirm, false);
    }

    public static void danger(Context context, String title, String message, Runnable onConfirm) {
        show(context, title, message, onConfirm, true);
    }

    private static void show(Context context, String title, String message, Runnable onConfirm, boolean danger) {
        java.util.Objects.requireNonNull(onConfirm);
        View view = LayoutInflater.from(context).inflate(R.layout.modal_status, null);
        TextView icon = view.findViewById(R.id.modal_icon);
        icon.setText(danger ? "!" : "?");
        icon.setBackgroundResource(danger ? R.drawable.status_danger_circle : R.drawable.status_success_circle);
        view.setBackgroundResource(danger ? R.drawable.danger_card : R.drawable.modal_card);
        ((TextView) view.findViewById(R.id.modal_title)).setText(title);
        ((TextView) view.findViewById(R.id.modal_message)).setText(message);
        Dialog modal = new Dialog(context);
        modal.requestWindowFeature(Window.FEATURE_NO_TITLE);
        modal.setContentView(view);
        TextView confirm = view.findViewById(R.id.modal_confirm);
        confirm.setText(R.string.confirm_arrow);
        confirm.setBackgroundResource(danger ? R.drawable.modal_danger : R.drawable.modal_confirm);
        confirm.setOnClickListener(v -> {
            confirm.setEnabled(false);
            modal.dismiss();
            onConfirm.run();
        });
        view.findViewById(R.id.modal_cancel).setOnClickListener(v -> modal.dismiss());
        modal.show();
        Window window = modal.getWindow();
        window.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        window.addFlags(WindowManager.LayoutParams.FLAG_DIM_BEHIND);
        window.setDimAmount(0.55f);
        float dp = context.getResources().getDisplayMetrics().density;
        int width = context.getResources().getDisplayMetrics().widthPixels;
        window.setLayout(Math.min(width - Math.round(48 * dp), Math.round(420 * dp)),
                WindowManager.LayoutParams.WRAP_CONTENT);
        icon.animate().scaleX(1).scaleY(1).setDuration(1200)
                .setInterpolator(new OvershootInterpolator()).start();
    }
}
```

### `app/src/main/java/com/example/app/helpers/AppToast.java`

```java
package com.example.app.helpers;

import android.app.Activity;
import android.os.Build;
import android.view.Gravity;
import android.view.View;
import android.view.WindowInsets;
import android.widget.FrameLayout;
import android.widget.TextView;
import com.example.app.R;

public final class AppToast {
    public static void success(Activity activity, String title, String message) {
        show(activity, title, message, false);
    }

    public static void error(Activity activity, String title, String message) {
        show(activity, title, message, true);
    }

    private static void show(Activity activity, String title, String message, boolean error) {
        FrameLayout root = activity.findViewById(android.R.id.content);
        View previous = root.findViewWithTag(AppToast.class);
        if (previous != null) {
            previous.animate().cancel();
            root.removeView(previous);
        }
        View banner = activity.getLayoutInflater().inflate(R.layout.toast_banner, root, false);
        banner.setTag(AppToast.class);
        banner.setBackgroundResource(error ? R.drawable.danger_card : R.drawable.modal_card);
        TextView icon = banner.findViewById(R.id.toast_icon);
        icon.setText(error ? "!" : "✓");
        icon.setBackgroundResource(error ? R.drawable.status_danger_circle : R.drawable.status_success_circle);
        ((TextView) banner.findViewById(R.id.toast_title)).setText(title);
        ((TextView) banner.findViewById(R.id.toast_message)).setText(message);
        int gap = Math.round(12 * activity.getResources().getDisplayMetrics().density);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(
                FrameLayout.LayoutParams.MATCH_PARENT, FrameLayout.LayoutParams.WRAP_CONTENT, Gravity.TOP);
        params.setMargins(gap, gap, gap, 0);
        banner.setOnApplyWindowInsetsListener((v, insets) -> {
            int top, left, right;
            if (Build.VERSION.SDK_INT >= 30) {
                android.graphics.Insets bars = insets.getInsets(
                        WindowInsets.Type.systemBars() | WindowInsets.Type.displayCutout());
                top = bars.top;
                left = bars.left;
                right = bars.right;
            } else {
                top = insets.getSystemWindowInsetTop();
                left = insets.getSystemWindowInsetLeft();
                right = insets.getSystemWindowInsetRight();
            }
            int[] position = new int[2];
            root.getLocationOnScreen(position);
            params.setMargins(gap + left, gap + Math.max(0, top - position[1]), gap + right, 0);
            banner.setLayoutParams(params);
            return insets;
        });
        root.addView(banner, params);
        banner.requestApplyInsets();
        banner.setTranslationY(-gap * 4);
        banner.setAlpha(0);
        banner.animate().translationY(0).alpha(1).setDuration(300).start();
        banner.setOnClickListener(v -> root.removeView(banner));
        banner.postDelayed(() -> banner.animate().alpha(0).translationY(-gap * 2).setDuration(200)
                .withEndAction(() -> root.removeView(banner)).start(), 4000);
    }
}
```

### `app/src/main/res/layout/modal_status.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="24dp"
    android:background="@drawable/modal_card">
    <TextView
        android:id="@+id/modal_icon"
        android:layout_width="96dp"
        android:layout_height="96dp"
        android:textColor="#FFFFFF"
        android:textSize="36sp"
        android:textStyle="bold"
        android:gravity="center"
        android:includeFontPadding="false"
        android:scaleX="0"
        android:scaleY="0"
        tools:background="@drawable/status_success_circle"
        tools:scaleX="1"
        tools:scaleY="1"
        tools:text="?" />
    <TextView
        android:id="@+id/modal_title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:gravity="center"
        android:textSize="22sp"
        android:textStyle="bold"
        android:textColor="#202033"
        tools:text="Confirm your action" />
    <TextView
        android:id="@+id/modal_message"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:gravity="center"
        android:textSize="15sp"
        android:lineSpacingExtra="3dp"
        android:textColor="#6B6B80"
        tools:text="Are you sure you want to continue? Confirm to complete this action." />
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="28dp"
        android:orientation="horizontal">
        <Button
            android:id="@+id/modal_cancel"
            android:layout_width="0dp"
            android:layout_height="52dp"
            android:layout_weight="1"
            android:layout_marginEnd="12dp"
            android:background="@drawable/modal_cancel"
            android:backgroundTint="@null"
            android:stateListAnimator="@null"
            android:text="@android:string/cancel"
            android:textColor="#29293D"
            android:textSize="15sp"
            android:textAllCaps="false" />
        <Button 
            android:id="@+id/modal_confirm"
            android:layout_width="0dp"
            android:layout_height="52dp"
            android:layout_weight="1"
            android:background="@drawable/modal_confirm"
            android:backgroundTint="@null"
            android:elevation="6dp"
            android:textColor="#FFFFFF"
            android:textSize="15sp"
            android:textAllCaps="false"
            tools:text="@string/confirm_arrow" />
    </LinearLayout>
</LinearLayout>
```

### `app/src/main/res/layout/toast_banner.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:gravity="center_vertical"
    android:padding="16dp"
    android:elevation="8dp"
    tools:background="@drawable/modal_card">
    <TextView
        android:id="@+id/toast_icon"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:gravity="center"
        android:includeFontPadding="false"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="#FFFFFF"
        tools:background="@drawable/status_success_circle"
        tools:text="✓" />
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:layout_marginStart="12dp"
        android:orientation="vertical">
        <TextView
            android:id="@+id/toast_title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="16sp"
            android:textStyle="bold"
            android:textColor="#202033"
            tools:text="Changes saved" />
        <TextView
            android:id="@+id/toast_message"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="4dp"
            android:textSize="14sp"
            android:textColor="#6B6B80"
            tools:text="Your changes have been saved successfully." />
    </LinearLayout>
</LinearLayout>
```

### `app/src/main/res/values/colors.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color
        name="black">#FF000000</color>
    <color
        name="white">#FFFFFFFF</color>
    <color
        name="success_pale">#E3F8F0</color>
    <color
        name="success_ring">#C4F1DF</color>
    <color
        name="success_light">#36D399</color>
    <color
        name="success">#16B981</color>
    <color
        name="badge_green_text">#12805C</color>
    <color
        name="badge_red_text">#BC3048</color>
    <color
        name="badge_blue_text">#285FB9</color>
    <color
        name="badge_yellow_text">#946900</color>
    <color
        name="badge_purple_text">#7944AD</color>
</resources>
```

### `app/src/main/res/values/component_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string
        name="confirm_arrow">Confirm →</string>
</resources>
```

### `app/src/main/res/drawable/modal_card.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="24dp" />
    <gradient
        android:angle="225"
        android:startColor="@color/success_pale"
        android:centerColor="#F7FCFA"
        android:endColor="#FFFFFF" />
</shape>
```

### `app/src/main/res/drawable/danger_card.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="24dp" />
    <gradient
        android:angle="225"
        android:startColor="#FFE3E7"
        android:centerColor="#FFF8F9"
        android:endColor="#FFFFFF" />
</shape>
```

### `app/src/main/res/drawable/modal_confirm.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#40FFFFFF">
    <item>
        <shape>
            <corners
                android:radius="12dp" />
            <gradient
                android:angle="270"
                android:startColor="@color/success_light"
                android:endColor="@color/success" />
        </shape>
    </item>
</ripple>
```

### `app/src/main/res/drawable/modal_danger.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#40FFFFFF">
    <item>
        <shape>
            <corners
                android:radius="12dp" />
            <gradient
                android:angle="270"
                android:startColor="#EF4258"
                android:endColor="#D91F36" />
        </shape>
    </item>
</ripple>
```

### `app/src/main/res/drawable/modal_cancel.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#1816B981">
    <item>
        <shape>
            <corners
                android:radius="12dp" />
            <solid
                android:color="#60FFFFFF" />
            <stroke
                android:width="1dp"
                android:color="#DADDEA" />
        </shape>
    </item>
</ripple>
```

### `app/src/main/res/drawable/status_success_circle.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape
            android:shape="oval">
            <solid
                android:color="@color/success_pale" />
        </shape>
    </item>
    <item
        android:left="6dp"
        android:top="6dp"
        android:right="6dp"
        android:bottom="6dp">
        <shape
            android:shape="oval">
            <solid
                android:color="@color/success_ring" />
        </shape>
    </item>
    <item
        android:left="13dp"
        android:top="13dp"
        android:right="13dp"
        android:bottom="13dp">
        <shape
            android:shape="oval">
            <gradient
                android:angle="270"
                android:startColor="@color/success_light"
                android:endColor="@color/success" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/drawable/status_danger_circle.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape
            android:shape="oval">
            <solid
                android:color="#FFF0F1" />
        </shape>
    </item>
    <item
        android:left="6dp"
        android:top="6dp"
        android:right="6dp"
        android:bottom="6dp">
        <shape
            android:shape="oval">
            <solid
                android:color="#FFDADD" />
        </shape>
    </item>
    <item
        android:left="13dp"
        android:top="13dp"
        android:right="13dp"
        android:bottom="13dp">
        <shape
            android:shape="oval">
            <gradient
                android:angle="270"
                android:startColor="#F65A6C"
                android:endColor="#DC263D" />
        </shape>
    </item>
</layer-list>
```

## 6. Complete badge files and usage

The drawable contains the pill and dot; the style supplies size and text color. Set BOTH background and text color when changing a recycled row from one color to another. Do not tint the whole layered background, because that would also recolor the dot/gradient.

### `app/src/main/res/drawable/green_badge.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:paddingMode="stack">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#F1FCF8"
                android:endColor="#E6FAF2" />
            <stroke
                android:width="1dp"
                android:color="#D7F6EC" />
            <padding
                android:left="28dp"
                android:right="12dp"
                android:top="6dp"
                android:bottom="6dp" />
        </shape>
    </item>
    <item
        android:width="8dp"
        android:height="8dp"
        android:left="12dp"
        android:gravity="left|center_vertical">
        <shape
            android:shape="oval">
            <solid
                android:color="#10BE8B" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/drawable/red_badge.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:paddingMode="stack">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#FFF3F5"
                android:endColor="#FFE8ED" />
            <stroke
                android:width="1dp"
                android:color="#FBD5DD" />
            <padding
                android:left="28dp"
                android:right="12dp"
                android:top="6dp"
                android:bottom="6dp" />
        </shape>
    </item>
    <item
        android:width="8dp"
        android:height="8dp"
        android:left="12dp"
        android:gravity="left|center_vertical">
        <shape
            android:shape="oval">
            <solid
                android:color="#EF4260" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/drawable/blue_badge.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:paddingMode="stack">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#EFF6FF"
                android:endColor="#E2EEFF" />
            <stroke
                android:width="1dp"
                android:color="#D8E7FF" />
            <padding
                android:left="28dp"
                android:right="12dp"
                android:top="6dp"
                android:bottom="6dp" />
        </shape>
    </item>
    <item
        android:width="8dp"
        android:height="8dp"
        android:left="12dp"
        android:gravity="left|center_vertical">
        <shape
            android:shape="oval">
            <solid
                android:color="#4385F5" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/drawable/yellow_badge.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:paddingMode="stack">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#FFFBEB"
                android:endColor="#FFF3CE" />
            <stroke
                android:width="1dp"
                android:color="#FAE9AB" />
            <padding
                android:left="28dp"
                android:right="12dp"
                android:top="6dp"
                android:bottom="6dp" />
        </shape>
    </item>
    <item
        android:width="8dp"
        android:height="8dp"
        android:left="12dp"
        android:gravity="left|center_vertical">
        <shape
            android:shape="oval">
            <solid
                android:color="#E8B129" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/drawable/purple_badge.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:paddingMode="stack">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#F8F2FF"
                android:endColor="#EEDFFF" />
            <stroke
                android:width="1dp"
                android:color="#E8D9FC" />
            <padding
                android:left="28dp"
                android:right="12dp"
                android:top="6dp"
                android:bottom="6dp" />
        </shape>
    </item>
    <item
        android:width="8dp"
        android:height="8dp"
        android:left="12dp"
        android:gravity="left|center_vertical">
        <shape
            android:shape="oval">
            <solid
                android:color="#9A62DE" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/values/badge_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style
        name="StatusBadge"
        parent="">
        <item
            name="android:layout_width">wrap_content</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:minHeight">28dp</item>
        <item
            name="android:gravity">center_vertical</item>
        <item
            name="android:includeFontPadding">false</item>
        <item
            name="android:textSize">12sp</item>
        <item
            name="android:textStyle">bold</item>
    </style>
    <style
        name="green_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/green_badge</item>
        <item
            name="android:textColor">@color/badge_green_text</item>
    </style>
    <style
        name="red_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/red_badge</item>
        <item
            name="android:textColor">@color/badge_red_text</item>
    </style>
    <style
        name="blue_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/blue_badge</item>
        <item
            name="android:textColor">@color/badge_blue_text</item>
    </style>
    <style
        name="yellow_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/yellow_badge</item>
        <item
            name="android:textColor">@color/badge_yellow_text</item>
    </style>
    <style
        name="purple_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/purple_badge</item>
        <item
            name="android:textColor">@color/badge_purple_text</item>
    </style>
</resources>
```

Static usage, with any label:

```xml
<TextView
    style="@style/green_badge"
    android:text="Active" />
```

Dynamic row binding:

```java
boolean active = model.isActive == 1;
holder.status.setText(active ? "Active" : "Inactive");
holder.status.setBackgroundResource(active ? R.drawable.green_badge : R.drawable.red_badge);
holder.status.setTextColor(context.getColor(active ? R.color.badge_green_text : R.color.badge_red_text));
```

The text color is deliberately separate from the background. A drawable cannot set a TextView's text color. Changing only the background is insufficient.

Material `ChipGroup` is allowed for automatic wrapping when Material Components is already available locally. Do not download it to add wrapping. Follow the exact list/chip design and icon rules in sections 18–19.

## 7. Models and static session: complete files

Use public fields and `TABLE_NAME`. The implicit public no-argument constructor supports reflection. SQL columns must match the fields selected by DBHelper. Do not invent tables just because a model exists. Fields shown here are sample data, not a required domain schema for every application.

### `app/src/main/java/com/example/app/models/User.java`

```java
package com.example.app.models;

public class User {
    public static final String TABLE_NAME = "users";

    public int id, age, creditPoints, isActive;
    public String image, fullName, username, password, phone, gender, role, specialty;
    public String email, location, joinedDate;
}
```

### `app/src/main/java/com/example/app/models/StaticUserModel.java`

```java
package com.example.app.models;

public class StaticUserModel {
    public static User user;
}
```

Set `StaticUserModel.user` after a successful login. This is an in-memory session and is lost when the process ends. Do not add preferences, tokens, encryption libraries, or a SessionHelper unless requested.

For logout: show the confirmation modal, set `StaticUserModel.user = null` only on Confirm, show a success toast, then navigate to login with `Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK`. Use the real login Activity for that project. If immediate navigation prevents the toast from being seen, show that logout result in the destination Activity. Do not allow Back to reopen authenticated screens.

## 8. DBHelper: exact reflection-based format

Use `helpers.DBHelper extends android.database.sqlite.SQLiteOpenHelper`. No Room, ORM, repository, dependency injection, or external serialization library.

- The reference uses `test.db`, version 2. Preserve the destination project's database name/version; update the version only for a requested schema migration.
- The blank `onCreate`/`onUpgrade` below belongs only to this generic reference project, which has no specified domain schema. For a real application or machine problem, define its required tables in `onCreate`, then seed ALL created tables in dependency order. Include supporting entities needed by the workflows even if the problem gives no example records. For later schema changes, migrate and seed new tables in `onUpgrade` without dropping user data.
- Read table names from public static `TABLE_NAME`.
- Public fields map to columns. Supported stored types are String, int/Integer, double/Double. Static fields and `id` are skipped on writes. Use plain integer ID fields such as `userId`, `studentId`, or `subjectId` for relationships.
- Keep the supplied `finalGrade` → `final` alias only if preserving this exact helper; do not add arbitrary model-specific aliases.
- NEVER declare SQLite foreign keys: no `FOREIGN KEY`, `REFERENCES`, cascading rules, or enforced table links. Store plain `INTEGER` columns named `userId`, `studentId`, `subjectId`, `orderId`, and so on. The model has matching `int` fields. Do not add related-object fields such as `User user` or automatic nested-object loading; this DBHelper reads scalar fields only. When a screen needs a related record, fetch it explicitly by its stored ID.
- A model needs a public no-argument constructor, including the implicit constructor when no other constructor is declared.
- Use parameterized selections for values. Close cursors with try-with-resources. Do not swallow failures or show a success toast when the result is false.
- Catch `ReflectiveOperationException` and relevant SQLite errors at the transaction boundary.
- Do not add float, boolean, long, date, enum, or custom object storage without deliberately extending the mapper when the feature needs it.

The following template is the original reflection helper format with only Android SDK and Java imports. Add schema and seed methods when implementing an actual problem. This blank template is not permission to omit required tables or seeders.

### `app/src/main/java/com/example/app/helpers/DBHelper.java`

```java
package com.example.app.helpers;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
import java.util.ArrayList;

public class DBHelper extends SQLiteOpenHelper {
    public DBHelper(Context context) {
        super(context, "test.db", null, 2);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {

    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }

    private String getTableName(Class<?> modelClass) throws ReflectiveOperationException {
        return (String) modelClass.getField("TABLE_NAME").get(null);
    }

    private boolean isColumnType(Class<?> type) {
        return type == String.class || type == int.class || type == Integer.class
                || type == double.class || type == Double.class;
    }

    private String getColumnName(Field field) {
        return field.getName().equals("finalGrade") ? "final" : field.getName();
    }

    private ContentValues toContentValues(Object model) throws IllegalAccessException {
        ContentValues cv = new ContentValues();

        for (Field field : model.getClass().getFields()) {
            if (Modifier.isStatic(field.getModifiers()) || field.getName().equals("id")
                    || !isColumnType(field.getType())) continue;

            String column = getColumnName(field);
            Object value = field.get(model);

            if (value == null) cv.putNull(column);
            else if (value instanceof String) cv.put(column, (String) value);
            else if (value instanceof Integer) cv.put(column, (Integer) value);
            else cv.put(column, (Double) value);
        }

        return cv;
    }

    private <T> T fromCursor(Cursor cursor, Class<T> modelClass) throws ReflectiveOperationException {
        T model = modelClass.getDeclaredConstructor().newInstance();

        for (Field field : modelClass.getFields()) {
            if (Modifier.isStatic(field.getModifiers()) || !isColumnType(field.getType())) continue;

            Class<?> type = field.getType();
            int column = cursor.getColumnIndexOrThrow(getColumnName(field));
            if (cursor.isNull(column) && !type.isPrimitive()) {
                field.set(model, null);
            } else if (type == String.class) {
                field.set(model, cursor.getString(column));
            } else if (type == int.class || type == Integer.class) {
                field.set(model, cursor.getInt(column));
            } else {
                field.set(model, cursor.getDouble(column));
            }
        }

        return model;
    }

    public boolean create(Object model) throws ReflectiveOperationException {
        return getWritableDatabase().insert(
                getTableName(model.getClass()), null, toContentValues(model)
        ) != -1;
    }

    public boolean update(Object model, int id) throws ReflectiveOperationException {
        return getWritableDatabase().update(
                getTableName(model.getClass()), toContentValues(model),
                "id = ?", new String[]{String.valueOf(id)}
        ) > 0;
    }

    public boolean delete(Class<?> modelClass, int id) throws ReflectiveOperationException {
        return getWritableDatabase().delete(
                getTableName(modelClass), "id = ?", new String[]{String.valueOf(id)}
        ) > 0;
    }

    public <T> T show(Class<T> modelClass, int id) throws ReflectiveOperationException {
        try (Cursor cursor = getReadableDatabase().query(
                getTableName(modelClass), null,
                "id = ?", new String[]{String.valueOf(id)}, null, null, null
        )) {
            return cursor.moveToFirst() ? fromCursor(cursor, modelClass) : null;
        }
    }

    public <T> ArrayList<T> get(Class<T> modelClass) throws ReflectiveOperationException {
        ArrayList<T> models = new ArrayList<>();

        try (Cursor cursor = getReadableDatabase().query(
                getTableName(modelClass), null, null, null, null, null, "id DESC"
        )) {
            while (cursor.moveToNext()) {
                models.add(fromCursor(cursor, modelClass));
            }
        }

        return models;
    }
}
```

Public calls stay short:

```java
DBHelper db = new DBHelper(context);
boolean created = db.create(user);
boolean updated = db.update(user, user.id);
boolean deleted = db.delete(User.class, user.id);
User found = db.show(User.class, user.id);
ArrayList<User> users = db.get(User.class);
```

These calls throw checked reflection errors and require tables to exist. The example above illustrates the signatures; invoke them inside the guarded transaction pattern below, not as unhandled statements.

## 9. Transaction flow: mandatory modal and result toast

Transactions are operations that change application/domain state: create, update, delete, cancel, submit, login/logout when applicable, or change a stored status. Opening a page, expanding a menu, displaying records, and pressing Cancel are not completed transactions.

1. Validate the input relevant to the action.
2. Show `AppModal.confirm` for a normal state change or `AppModal.danger` for a destructive one.
3. Run the mutation ONLY inside the modal's Confirm action.
4. Use the actual result: `AppToast.success` on success and `AppToast.error` on failure/exception.
5. Refresh the affected list/screen only after success.
6. Do not add a second confirmation, success modal, or fake progress delay.

`AppModal` takes an Activity-backed Context. `AppToast` takes an Activity because the banner attaches to that screen. Do not pass an application Context or store an Activity statically. The Context/list-only rule applies to adapter constructors, not these helper methods.

Example Activity method, using the exact helpers above:

```java
private void deleteUser(User user) {
    AppModal.danger(this, "Delete user?", "This action cannot be undone.", () -> {
        try (DBHelper db = new DBHelper(this)) {
            if (db.delete(User.class, user.id)) {
                AppToast.success(this, "User deleted", "The user was removed.");
                refreshUsers();
            } else {
                AppToast.error(this, "Couldn’t delete user", "Please try again.");
            }
        } catch (ReflectiveOperationException | android.database.sqlite.SQLiteException error) {
            AppToast.error(this, "Couldn’t delete user", "Please try again.");
        }
    });
}
```

`refreshUsers()` means that screen's real data-loading method. Define it for the screen rather than leaving an unresolved call. Do not add a database deletion when the user asked only for an in-memory preview. Even an intentional preview mutation gets a modal and result toast; clearly label it as preview data.

Normal confirmation and standalone feedback calls:

```java
AppModal.confirm(context, "Save changes?", "Confirm to save your changes.", () -> saveUser());
AppToast.success(activity, "Changes saved", "Your changes were saved successfully.");
AppToast.error(activity, "Couldn’t save", "Please try again.");
```

`saveUser()` is the feature's real operation; it must follow the same result/error handling as `deleteUser`. Literal text is used here for clarity. Put reusable/runtime labels in string resources in the actual application, and do not manufacture dozens of extra strings for unspecified features.

## 10. RecyclerView adapter rules and concrete structure

Use RecyclerView/LinearLayoutManager from the locally available AndroidX dependencies. They are allowed components, not an exception to an SDK-only rule. Prefer the dependency/version already in the destination project. Do not introduce a paging library, custom adapter DSL, or listener architecture for a small sample list unless the requested behavior needs it.

- Model-specific adapter, a separate row XML, and a ViewHolder.
- Fields follow `public Context context; public ArrayList<User> list;`.
- Constructor is EXACTLY `UserAdapter(Context context, ArrayList<User> list)`.
- Required methods are `onCreateViewHolder`, `onBindViewHolder`, `getItemCount`, and a ViewHolder constructor that binds row IDs.
- Put row navigation and transaction handling inside the adapter. Pass a model ID via Intent to a real edit/detail Activity. Do not pass lambdas/consumers into the constructor or move the same callback to a setter as a workaround.
- Refresh the hosting screen or update the list with the appropriate adapter notification after successful operations.
- Reset visibility, image, text, badge background, and text color on EVERY bind when those values vary.
- Keep passwords and other private credentials out of rows.
- Do not create fake tables or edits for preview data.

Constructor and creation must look like this:

```java
public UserAdapter(Context context, ArrayList<User> list) {
    this.context = context;
    this.list = list;
}
```

```java
RecyclerView recycler = findViewById(R.id.users_list);
recycler.setLayoutManager(new LinearLayoutManager(this));
recycler.setAdapter(new UserAdapter(this, users));
```

The complete approved person, item and activity row XML, bound IDs, drawables and styles appear in section 19. Use the correct one for the record type and preserve its visual design.

## 11. Activity theme and offline component integration

The helper templates need no custom Material theme overlay. A new SDK-only Activity can use the platform `Theme.Material.Light.NoActionBar` or its appropriate native theme. Material Components themes and AppCompat Activities are also allowed when their libraries are already available locally. Keep the existing project setup and theme compatible rather than rewriting them unnecessarily. The Android framework Theme.Material and the Material Components library are different options, both acceptable when usable offline.

Example theme for a fresh native setup:

```xml
<resources>
    <style
        name="AppTheme"
        parent="android:style/Theme.Material.Light.NoActionBar">
        <item
            name="android:fontFamily">sans</item>
        <item
            name="android:colorAccent">@color/success</item>
        <item
            name="android:windowLightStatusBar">true</item>
    </style>
</resources>
```

The modal template explicitly supplies its dimensions, backgrounds, buttons, and dim amount, so it does not rely on a missing external theme/style. Use the Activity's normal content root for the toast. Keep status-bar/inset handling appropriate to the target SDK; do not remove it merely to save lines if that puts controls behind system bars.

## 12. List XML: deliberately pending

Do NOT define new detailed rules for `list.xml` or model-specific row files yet. The user will go through these deeply later. General one-attribute-per-line formatting, reuse of the documented badges, and the offline-only requirement already apply. Do not add fields, menus, chip wrapping, or particular card layouts based on an unavailable screenshot. Obtain the user's actual list requirements when that work begins.

## 13. Verification and handoff

- Build after implementation changes with `./gradlew --offline :app:assembleDebug` or the destination project's existing offline equivalent. Never silently retry an offline failure with network access enabled.
- For meaningful UI/transaction changes, inspect an available emulator/device and verify the action, Cancel behavior, success/error feedback, and list refresh as applicable.
- Do not add tests that only mirror a trivial layout. Do not run unrelated broad checks after the relevant checks have passed.
- Documentation-only changes do not require rebuilding the existing application. Embedded code templates should be checked for missing resources, offline dependency availability, valid XML, and compilation when practical. The platform-only helper templates must remain usable without additional UI libraries.
- In the response, state what changed, what was verified, and any real limitation. Never claim a sample operation updated a database when it only changed an in-memory list.
- Keep everything needed to transport these rules IN THIS FILE. Do not split instructions into another README, depend on a screenshot attachment, or reference another computer's filesystem for component implementations.

## 14. Mandatory Activity naming and separation

Every role-specific Activity must follow `<Role><ScreenOrAction>Activity` in PascalCase and live in that role's package. Use the correctly spelled role and screen names, with no spaces in class names.

| Role | Package | Dashboard Activity | Dashboard layout |
| --- | --- | --- | --- |
| Admin | `admin` | `AdminDashboardActivity` | `activity_admin_dashboard.xml` |
| Customer | `customer` | `CustomerDashboardActivity` | `activity_customer_dashboard.xml` |
| Instructor | `instructor` | `InstructorDashboardActivity` | `activity_instructor_dashboard.xml` |

**NEVER share one Activity between roles or distinct screen actions.** Similar appearances, fields, or logic do not justify combining screens.

- Each role gets its own dashboard Activity and screen layout. Do not create one `DashboardActivity` that switches behavior based on the logged-in user's role.
- Create and update are separate Activities and separate screen layouts. Do not use an ID, Intent extra, boolean, mode, or conditional to turn one Activity into both a create screen and an update screen.
- Detail/show screens are separate for each role. An admin's order detail screen must not be the same Activity or screen layout as a customer's order detail screen.
- List/manage screens are separate per role as well. Do not combine admin management and customer viewing into one Activity with hidden controls.
- Route explicitly to the correct role-specific Activity. Do not use a generic router Activity as the shared implementation of these screens.
- Do not evade this rule by using one shared base Activity, Fragment, or included full-screen layout that contains the role-specific screen behavior and merely adding thin role wrappers.
- Shared `DBHelper`, models, modal/toast helpers, badges, drawables, and small UI components remain reusable. This reuse must not merge screen ownership or actions.
- The requirement for separate screens takes precedence over reducing the number of files or lines. Keep each screen's implementation short without combining it with another screen.
- Register every created Activity in the manifest. Ordinary internal screens should not be exported unless an actual requested entry point requires it.
- Only create the screens requested by the user. This naming rule is not authorization to generate every role's screens preemptively.

Concrete examples:

| Purpose | Activity | Screen layout |
| --- | --- | --- |
| Admin creates a user | `AdminCreateUserActivity` | `activity_admin_create_user.xml` |
| Admin updates a user | `AdminUpdateUserActivity` | `activity_admin_update_user.xml` |
| Admin manages orders | `AdminOrderListActivity` | `activity_admin_order_list.xml` |
| Customer lists their orders | `CustomerOrderListActivity` | `activity_customer_order_list.xml` |
| Admin shows an order | `AdminShowOrderActivity` | `activity_admin_show_order.xml` |
| Customer shows an order | `CustomerShowOrderActivity` | `activity_customer_show_order.xml` |
| Instructor creates a class | `InstructorCreateClassActivity` | `activity_instructor_create_class.xml` |
| Instructor updates a class | `InstructorUpdateClassActivity` | `activity_instructor_update_class.xml` |

For example, `AdminShowOrderActivity` and `CustomerShowOrderActivity` can both receive an `orderId` and use the same Order model/DBHelper. They still own separate layouts, actions, and role-appropriate behavior. Likewise, `AdminCreateUserActivity` and `AdminUpdateUserActivity` must remain separate even when their forms have matching fields.

A truly role-independent entry screen such as login can retain a neutral name such as `LoginActivity`; do not duplicate it unless requested. This does not permit sharing authenticated role-specific screens.

## 15. Offline availability checklist

The user's meaning of “no third parties” is **Android Studio without internet**, not a ban on AndroidX, Material, or a library that is already available locally. This clarification supersedes earlier SDK-only wording.

Before using a component:

1. Inspect the destination project's Gradle configuration and installed SDK/build tools.
2. Reuse the existing dependency version. Confirm the artifact and all transitive dependencies/plugins are locally cached or bundled; merely seeing a component in an IDE palette or knowing its Maven coordinates does not establish offline availability.
3. Build with Gradle's `--offline` flag. A cached dependency is usable; an unresolved dependency must not trigger a download.
4. If unavailable, use an already available component or the Android SDK fallback. Report the missing local artifact instead of silently enabling internet or installing packages.
5. Verify the app's actual feature works with internet disabled. A successful offline compile does not prove that a runtime feature avoids network services.

Examples:

| Allowed when available locally | Not allowed in this offline workflow |
| --- | --- |
| RecyclerView, LinearLayoutManager, AppCompat, ConstraintLayout | Downloading a new AndroidX version |
| MaterialButton, MaterialCardView, MaterialAlertDialogBuilder, ChipGroup | Downloading Material Components to add a widget |
| Android SDK APIs, XML drawables, system fonts, local assets | Downloading icons, fonts, images, or loading remote image URLs |
| DBHelper with local SQLite, local files, local in-memory session | REST/GraphQL services, hosted databases, cloud authentication |
| A useful preinstalled/cached third-party library with no runtime network requirement | A library whose artifacts are missing or whose feature requires an online API |

No change to the requested Java/XML architecture, DBHelper format, adapter constructor rule, modal/toast flow, or role-separated Activities is implied by allowing locally available libraries. Keep implementations short and reuse the components the computer already has.

## 16. ItemCard convention

The generic item-card component is named **ItemCard**, implemented by `res/layout/sample_item_card_list.xml`. Use it for products, foods, animals, and similar image-led item records. The user clarified that this must keep the original VERTICAL shopping-card design, scaled down into a small tile: photo above details, price/action below. Preview multiple tiles using a two-column RecyclerView grid. The vertical card must stay vertical and compact. A separate `horizontal_sample_item_card_list.xml` variant is also kept and approved by the user; preserve both variants. Do not use an oversized full-screen hero card. The size fix belongs in sample_item_card_list.xml itself, not only in MainActivity. It is distinct from the generic person row `sample_person_list.xml`. The sample `Item` model and `ItemAdapter(Context context, ArrayList<Item> list)` demonstrate binding without authorizing an items table. All preview photos are local assets; never use remote image URLs. Complete portable layout XML and supporting resources are embedded in section 19.

## 17. ActivityCard convention

The generic **ActivityCard** layout is `res/layout/sample_activity_card_list.xml`, intended for appointments, invitations, requests, orders, payments, bookings, and similar activity/transaction records. It is a compact white/lavender card with a leading type icon, title and subtitle, colored type/status badge, wrapping date/time/mode chips, organizer identity, overflow button, and primary/secondary actions. Invitation placeholders use purple, Decline, and Accept. Bind actual labels, colors, metadata, and actions for the record type; do not assume every record is an invitation. Mutating actions still require AppModal confirmation and AppToast result feedback. The sample XML is a visual template, not authorization to create tables, backend operations, or additional Activities. Complete portable layout XML and supporting resources are embedded in section 19.

## 18. Role-specific list screen naming and layout

Name every list Activity `RoleListNameActivity`, where ListName includes the entity followed by `List`: `AdminMenuListActivity`, `CustomerOrderListActivity`, `InstructorAppointmentListActivity`. Use the correctly spelled `Activity` suffix. Put the class in its role package, such as `admin`, and name its dedicated layout `activity_admin_menu_list.xml`. This rule supersedes earlier list names such as AdminManageOrdersActivity or CustomerOrdersActivity. Never share a list Activity or its full-screen layout between roles.

EVERY `*ListActivity` MUST follow this same screen design and vertical order, across all roles and record types, unless the user explicitly requests a different design. This is the required list-screen template, not merely a menu example. Adapt the title, search hint, category labels/icons, counts, and row content to the entity. Keep the shared visual structure consistent.

Use a white screen, back arrow, centered bold entity title, and green rounded Add button when that role can create records. Omit Add when creation is unavailable to that role; never show an unauthorized or fake action. The menu example uses Menu Items and Add Item. Beneath the header, place a pale gray rounded search field with a search icon and a separate outlined circular filter/sort button. Below that use a wrapping Material ChipGroup with singleSelection and selectionRequired. Always include All; use categories or statuses relevant to the list. Menu examples are Food, Drinks, Desserts, Breakfast, Lunch, Dinner. Do not copy food categories into unrelated lists. Each chip MUST have a local vector icon that depicts its actual label or category: a grid for All, fork/food for Food, cup for Drinks, leaf for Desserts, sun for Breakfast, serving dish for Lunch, and moon for Dinner. For other lists choose equally specific icons, such as person for Users, box for Products, paw for Pets, calendar for Appointments, receipt for Orders, or payment card for Payments. Draw a small matching Android vector when one is missing; do not reuse an unrelated icon merely because it already exists. Each chip has a label and count. Selected chips are pale green with dark green text/icons; unselected chips are pale gray with dark text/icons. Counts must reflect the actual sample/data set, never copy unrelated screenshot totals. Count the same collection that the screen displays.

Place a RecyclerView directly BELOW the ChipGroup, filling the remaining height with layout_height="0dp" and layout_weight="1" in a vertical LinearLayout. Do not replace it with repeated static rows or place it inside a ScrollView. Use the appropriate reusable row; the menu example uses a two-column grid of compact sample_item_card_list tiles. Search and category filters update the adapter list without changing newest-first order. Newly created records appear at the TOP immediately after a successful create, including when returning from a create screen. Show an empty-state message when there are no matching records. Keep adapter constructors limited to Context and list. Actual create/update actions use separate role-specific screens and retain modal/toast transaction rules; a visual sample does not authorize a database schema or a full create form.

Portable visual measurements for every ListActivity: use system sans-serif, 16dp horizontal screen padding and status/navigation-bar insets. Header is 64dp high; back icon button is 40dp, title is bold 22sp in #101A23, Add button is 44dp high with 100dp corners, #08783F background, white 12sp text and an 18dp plus icon. The search/filter row is 52dp high, 12dp below the header; search fills remaining width, uses #F5F6F8 background with 100dp corners, 18dp horizontal padding, 24dp search icon, 12dp drawable spacing, 14sp text and #8B929E hint. The filter button is a 52dp white circle with a 1dp #EEF0F3 outline, 10dp start gap and a green local vector filter icon. ChipGroup sits 16dp below search, with 6dp horizontal and 2dp vertical chip spacing and 8dp bottom margin. Chips wrap naturally, have 40dp minimum height, 100dp corners, 12sp text, 20dp local icons and no stroke. Selected colors are #E6F7EE background and #08783F text/icon; unselected colors are #F5F6F8 background and #18232F text/icon. RecyclerView fills the remaining screen, clips no bottom padding and has 16dp bottom padding. Choose the approved row layout for the entity: person rows and activity cards use a vertical LinearLayoutManager; compact vertical item tiles use a two-column GridLayoutManager. Keep all XML attributes on separate lines and use existing locally available components without adding downloads or unnecessary layers.

## 19. Required list type mapping and complete portable XML

Choose the row by what each record represents, not by the screen title or the role. These mappings are mandatory unless the user explicitly requests another design. The exact section 19 XML is the implementation source of truth, not loose inspiration:

| Record type | Required row layout | RecyclerView layout |
| --- | --- | --- |
| People: users, customers, staff, admins, trainers, instructors, members | `sample_person_list.xml` | Vertical LinearLayoutManager |
| Items: products, foods, animals, pets, dogs, cats, inventory, and similar image-led records | `sample_item_card_list.xml` | Two-column GridLayoutManager with compact vertical tiles |
| Appointments, invitations, requests, orders, payments, bookings, and similar events/transactions | `sample_activity_card_list.xml` | Vertical LinearLayoutManager |

For example, a dog catalogue uses ItemCard, but a veterinary appointment for a dog uses ActivityCard. A customer directory uses PersonCard, but that customer's orders use ActivityCard. Every list loads the latest created data first: for SQLite use `ORDER BY createdAt DESC, id DESC` when a reliable creation timestamp exists, otherwise `ORDER BY id DESC`. Bind the ordered result to RecyclerView and preserve this order after filtering and refreshing; do not display an unsorted query or stale seed data after a successful create. The approved `horizontal_sample_item_card_list.xml` is an optional item-row variant when the requested screen calls for horizontal cards. Preserve it; do not silently replace the default vertical item tiles with horizontal rows.

Every list screen still uses the section 18 header → search/filter → wrapping category/status ChipGroup → RecyclerView structure. Adapt labels and visible metadata to the domain. Animal cards must show relevant animal details rather than calories/prices copied blindly from the food placeholder. An order card must show order status and order actions rather than invitation labels. Only show applicable actions and fields; keep the same visual design and bind actual data. Preserve the row hierarchy, dimensions, corner radii, margins, padding, borders, palette, typography, icon positions and button placement from the approved XML. Do not replace the RecyclerView row with a newly invented card, alter the person/item/activity card structure, or use a different drawable/style just because it is easier. Adapt labels, local images, badges and icon meanings while keeping the approved look. If a field does not apply, hide its existing view and rebalance only the necessary spacing; do not silently redesign the entire row. Render the real RecyclerView on an emulator or Android Studio preview and compare with the embedded XML/visual measurements before finishing.

### Appearance to reproduce

PersonCard: white rounded card with a faint mint/white gradient, 20dp corners, 1dp #DDF2EA border, 18dp internal padding and 8dp vertical spacing. Top row has a 56dp circular photo or initials fallback, bold 18sp name, gray 13sp role and colored status badge. Under it, metadata wraps as 11sp gray/mint pills with local vector icons, alongside a 48dp arrow/action button. Preserve wrapping; do not stretch the card into a full-screen panel.

ItemCard: compact vertical shopping tile with 16dp corners, 1dp #E1ECE6 border and 6dp margins. Photo is above details and exactly 112dp high, center-cropped with 12dp image corners; overlay a small availability badge at top-left and a 28dp favorite button at top-right. Body padding is 10dp, name bold 14sp, category gray 11sp, metadata pills and rating/location 10sp. Footer has bold 16sp price and a 32dp green rounded View Details button. Keep the tile small enough for a two-column grid; never use a full-screen hero photo/card.

ActivityCard: compact white/lavender gradient card with 20dp corners, 1dp #EEF0F5 border, 16dp padding and 8dp vertical margins. Header uses a 48dp rounded lavender icon panel, bold 16sp title, gray 12sp subtitle and 32dp overflow button. Metadata wraps in a ChipGroup 14dp below the header, using a colored badge plus date, time and location/mode pills. Footer sits 16dp below metadata, with 34dp organizer avatar/initials, bold 12sp name, gray 10sp role, and two 36dp rounded buttons. Invitation examples use an outlined gray Decline button and purple gradient Accept button. Choose badges/actions appropriate to the actual record type.

### How to recreate these templates from this file alone

The following are complete XML files copied from the approved project templates, including supporting styles, strings, selectors, backgrounds and native vector icons. No original project or screenshot is required. Use the destination app's existing offline-available AndroidX/Material setup; all XML attributes remain one per line and source has no comments. Merge resources by name into existing values files. Do not duplicate resources already recreated from earlier sections; these blocks include shared resources again to make the list templates self-contained. Android framework references and locally available Material widget styles come from those installed libraries.

The food text, dates and initials are preview placeholders; replace them through adapter binding for actual records. The original burger photograph is a local bitmap and cannot be carried by this Markdown file. If a suitable local image is available, use it instead of the `sample_item_photo.xml` fallback below. Otherwise the included native vector picture placeholder makes the row compile and preview without downloading anything. Never define both a bitmap and XML drawable with the same resource name.

The screen XML is the AdminMenuListActivity example. Recreate a separately owned layout for each role/entity screen, adapting its title, search hint and chips while retaining the structure. Use a plain model with public fields, a RecyclerView adapter with ONLY Context and list constructor arguments, and inflate the selected row with `inflate(R.layout.sample_person_list, parent, false)` or the corresponding item/activity row. Bind every dynamic field and reset visibility, badges and icons on each bind to avoid recycled content. The XML alone does not implement transactions. Wire authorized mutations through AppModal before execution and AppToast after the actual result; keep SQLite work in DBHelper.

### `app/src/main/res/layout/sample_person_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="20dp"
    android:layout_marginVertical="8dp"
    app:cardCornerRadius="20dp"
    app:cardElevation="0dp"
    app:strokeWidth="1dp"
    app:strokeColor="#DDF2EA"
    app:cardBackgroundColor="#FFFFFF">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/modal_card"
        android:orientation="vertical"
        android:padding="18dp">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical">
            <FrameLayout
                android:layout_width="56dp"
                android:layout_height="56dp">
                <TextView
                    android:id="@+id/user_initials"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:background="@drawable/user_avatar_background"
                    android:gravity="center"
                    android:text="@string/sample_user_initials"
                    android:textSize="22sp"
                    android:textStyle="bold"
                    android:textColor="#08784F" />
                <com.google.android.material.imageview.ShapeableImageView
                    android:id="@+id/user_image"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:scaleType="centerCrop"
                    android:visibility="gone"
                    app:shapeAppearanceOverlay="@style/UserAvatarShape" />
            </FrameLayout>
            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginStart="16dp"
                android:orientation="vertical">
                <TextView
                    android:id="@+id/user_name"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@string/sample_user_name"
                    android:textColor="#182231"
                    android:textSize="18sp"
                    android:textStyle="bold" />
                <TextView
                    android:id="@+id/user_role"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:text="@string/sample_user_role"
                    android:textSize="13sp"
                    android:textColor="#687582" />
            </LinearLayout>
            <TextView
                android:id="@+id/user_status"
                style="@style/green_badge"
                android:layout_marginStart="8dp"
                android:text="@string/user_active" />
        </LinearLayout>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:orientation="horizontal"
            android:gravity="center_vertical">
            <com.google.android.material.chip.ChipGroup
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                app:chipSpacingHorizontal="8dp"
                app:chipSpacingVertical="10dp">
                <TextView
                    android:id="@+id/user_phone"
                    style="@style/UserDetail"
                    android:drawableStart="@drawable/ic_user_phone"
                    android:text="@string/sample_user_phone" />
                <TextView
                    android:id="@+id/user_credits"
                    style="@style/UserDetail.Green"
                    android:drawableStart="@drawable/ic_user_credits"
                    android:text="@string/sample_user_credits" />
                <TextView
                    android:id="@+id/user_age"
                    style="@style/UserDetail"
                    android:drawableStart="@drawable/ic_user_calendar"
                    android:text="@string/sample_user_age" />
                <TextView
                    android:id="@+id/user_location"
                    style="@style/UserDetail.Green"
                    android:drawableStart="@drawable/ic_user_location"
                    android:text="@string/sample_user_location" />
                <TextView
                    android:id="@+id/user_email"
                    style="@style/UserDetail"
                    android:drawableStart="@drawable/ic_user_email"
                    android:text="@string/sample_user_email" />
                <TextView
                    android:id="@+id/user_reference"
                    style="@style/UserDetail.Green"
                    android:drawableStart="@drawable/ic_user_id"
                    android:text="@string/sample_user_id" />
                <TextView
                    android:id="@+id/user_joined"
                    style="@style/UserDetail"
                    android:drawableStart="@drawable/ic_user_calendar"
                    android:text="@string/sample_user_joined" />
                <TextView
                    android:id="@+id/user_specialty"
                    style="@style/UserDetail.Green"
                    android:drawableStart="@drawable/ic_user_certificate"
                    android:text="@string/sample_user_specialty" />
            </com.google.android.material.chip.ChipGroup>
            <ImageButton
                android:id="@+id/user_actions"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:layout_marginStart="10dp"
                android:background="@drawable/user_detail_background"
                android:src="@drawable/ic_user_chevron" />
        </LinearLayout>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

### `app/src/main/res/layout/sample_item_card_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="6dp"
    android:layout_marginVertical="6dp"
    app:cardCornerRadius="16dp"
    app:cardElevation="0dp"
    app:strokeWidth="1dp"
    app:strokeColor="#E1ECE6"
    app:cardBackgroundColor="#FFFFFF"
    xmlns:tools="http://schemas.android.com/tools"
    tools:layout_width="180dp">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="112dp">
            <com.google.android.material.imageview.ShapeableImageView
                android:id="@+id/item_image"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:src="@drawable/sample_item_photo"
                app:shapeAppearanceOverlay="@style/ItemPhotoShape" />
            <ImageButton
                android:id="@+id/item_favorite"
                android:layout_width="28dp"
                android:layout_height="28dp"
                android:layout_gravity="top|end"
                android:layout_margin="5dp"
                android:padding="7dp"
                android:background="@drawable/user_avatar_background"
                android:backgroundTint="#F2F0ED"
                android:src="@drawable/ic_item_heart" />
            <TextView
                android:id="@+id/item_status"
                style="@style/green_badge"
                android:layout_gravity="top|start"
                android:text="@string/item_available"
                android:textSize="9sp"
                android:layout_marginStart="5dp"
                android:layout_marginTop="5dp" />
        </FrameLayout>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="10dp">
            <TextView
                android:id="@+id/item_name"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/sample_item_name"
                android:textSize="14sp"
                android:textStyle="bold"
                android:textColor="#10182B"
                android:maxLines="2"
                android:ellipsize="end" />
            <TextView
                android:id="@+id/item_category"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="2dp"
                android:text="@string/sample_item_category"
                android:textSize="11sp"
                android:textColor="#8191A1" />
            <com.google.android.material.chip.ChipGroup
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="6dp"
                app:chipSpacingHorizontal="6dp"
                app:chipSpacingVertical="4dp">
                <TextView
                    android:id="@+id/item_detail"
                    style="@style/ItemChip.Info"
                    android:drawableStart="@drawable/ic_item_food"
                    android:text="@string/sample_item_detail" />
                <TextView
                    android:id="@+id/item_tag"
                    style="@style/ItemChip.Tag"
                    android:drawableStart="@drawable/ic_item_leaf"
                    android:text="@string/sample_item_tag" />
            </com.google.android.material.chip.ChipGroup>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="6dp"
                android:orientation="horizontal"
                android:gravity="center_vertical">
                <TextView
                    android:id="@+id/item_rating"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:drawableStart="@drawable/ic_item_star"
                    android:drawablePadding="4dp"
                    android:gravity="center_vertical"
                    android:text="@string/sample_item_rating"
                    android:textSize="10sp"
                    android:textColor="#526375" />
                <View
                    android:layout_width="1dp"
                    android:layout_height="10dp"
                    android:layout_marginHorizontal="5dp"
                    android:background="#DDE5EA" />
                <TextView
                    android:id="@+id/item_location"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="@string/sample_item_location"
                    android:textSize="10sp"
                    android:textColor="#526375" />
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="6dp"
                android:orientation="horizontal"
                android:gravity="center_vertical">
                <TextView
                    android:id="@+id/item_price"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:layout_marginEnd="4dp"
                    android:text="@string/sample_item_price"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:textColor="#10182B" />
                <com.google.android.material.button.MaterialButton
                    android:id="@+id/item_view_details"
                    android:layout_width="wrap_content"
                    android:layout_height="32dp"
                    android:minWidth="0dp"
                    android:background="@drawable/item_action_background"
                    android:backgroundTint="@null"
                    app:backgroundTint="@null"
                    android:paddingHorizontal="8dp"
                    android:insetTop="0dp"
                    android:insetBottom="0dp"
                    android:text="@string/item_view_details"
                    android:textColor="#FFFFFF"
                    android:textSize="10sp"
                    android:textAllCaps="false"
                    app:icon="@drawable/ic_user_chevron"
                    app:iconTint="#FFFFFF"
                    app:iconGravity="textEnd"
                    app:iconPadding="2dp"
                    app:iconSize="12dp" />
            </LinearLayout>
        </LinearLayout>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

### `app/src/main/res/layout/horizontal_sample_item_card_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="16dp"
    android:layout_marginVertical="6dp"
    app:cardCornerRadius="16dp"
    app:cardElevation="0dp"
    app:strokeWidth="1dp"
    app:strokeColor="#E1ECE6"
    app:cardBackgroundColor="#FFFFFF">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp">
        <FrameLayout
            android:layout_width="96dp"
            android:layout_height="110dp">
            <com.google.android.material.imageview.ShapeableImageView
                android:id="@+id/item_image"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:src="@drawable/sample_item_photo"
                app:shapeAppearanceOverlay="@style/ItemPhotoShape" />
            <ImageButton
                android:id="@+id/item_favorite"
                android:layout_width="30dp"
                android:layout_height="30dp"
                android:layout_gravity="top|end"
                android:layout_margin="5dp"
                android:padding="7dp"
                android:background="@drawable/user_avatar_background"
                android:backgroundTint="#F2F0ED"
                android:src="@drawable/ic_item_heart" />
            <TextView
                android:id="@+id/item_status"
                style="@style/green_badge"
                android:layout_gravity="bottom|center_horizontal"
                android:layout_marginBottom="5dp"
                android:text="@string/item_available"
                android:textSize="9sp" />
        </FrameLayout>
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_marginStart="12dp"
            android:orientation="vertical">
            <TextView
                android:id="@+id/item_name"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/sample_item_name"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="#10182B" />
            <TextView
                android:id="@+id/item_category"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="3dp"
                android:text="@string/sample_item_category"
                android:textSize="12sp"
                android:textColor="#8191A1" />
            <com.google.android.material.chip.ChipGroup
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="7dp"
                app:chipSpacingHorizontal="6dp"
                app:chipSpacingVertical="4dp">
                <TextView
                    android:id="@+id/item_detail"
                    style="@style/ItemChip.Info"
                    android:drawableStart="@drawable/ic_item_food"
                    android:text="@string/sample_item_detail" />
                <TextView
                    android:id="@+id/item_tag"
                    style="@style/ItemChip.Tag"
                    android:drawableStart="@drawable/ic_item_leaf"
                    android:text="@string/sample_item_tag" />
            </com.google.android.material.chip.ChipGroup>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="7dp"
                android:orientation="horizontal"
                android:gravity="center_vertical">
                <TextView
                    android:id="@+id/item_rating"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:drawableStart="@drawable/ic_item_star"
                    android:drawablePadding="4dp"
                    android:gravity="center_vertical"
                    android:text="@string/sample_item_rating"
                    android:textSize="11sp"
                    android:textColor="#526375" />
                <View
                    android:layout_width="1dp"
                    android:layout_height="12dp"
                    android:layout_marginHorizontal="8dp"
                    android:background="#DDE5EA" />
                <TextView
                    android:id="@+id/item_location"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="@string/sample_item_location"
                    android:textSize="11sp"
                    android:textColor="#526375" />
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:orientation="horizontal"
                android:gravity="center_vertical">
                <TextView
                    android:id="@+id/item_price"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:layout_marginEnd="8dp"
                    android:text="@string/sample_item_price"
                    android:textSize="19sp"
                    android:textStyle="bold"
                    android:textColor="#10182B" />
                <com.google.android.material.button.MaterialButton
                    android:id="@+id/item_view_details"
                    android:layout_width="wrap_content"
                    android:layout_height="36dp"
                    android:minWidth="0dp"
                    android:background="@drawable/item_action_background"
                    android:backgroundTint="@null"
                    app:backgroundTint="@null"
                    android:paddingHorizontal="10dp"
                    android:insetTop="0dp"
                    android:insetBottom="0dp"
                    android:text="@string/item_view_details"
                    android:textColor="#FFFFFF"
                    android:textSize="11sp"
                    android:textAllCaps="false"
                    app:icon="@drawable/ic_user_chevron"
                    app:iconTint="#FFFFFF"
                    app:iconGravity="textEnd"
                    app:iconPadding="4dp"
                    app:iconSize="14dp" />
            </LinearLayout>
        </LinearLayout>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

### `app/src/main/res/layout/sample_activity_card_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="16dp"
    android:layout_marginVertical="8dp"
    app:cardCornerRadius="20dp"
    app:cardElevation="0dp"
    app:strokeWidth="1dp"
    app:strokeColor="#EEF0F5"
    app:cardBackgroundColor="#FFFFFF">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp"
        android:background="@drawable/activity_card_background">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="top">
            <ImageView
                android:id="@+id/activity_icon"
                android:layout_width="48dp"
                android:layout_height="48dp"
                android:padding="12dp"
                android:background="@drawable/activity_icon_background"
                android:src="@drawable/ic_user_email"
                android:tint="#8A3FF0" />
            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginStart="12dp"
                android:orientation="vertical">
                <TextView
                    android:id="@+id/activity_title"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@string/sample_activity_title"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:textColor="#111B2D" />
                <TextView
                    android:id="@+id/activity_subtitle"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:text="@string/sample_activity_subtitle"
                    android:textSize="12sp"
                    android:textColor="#8A99AD" />
            </LinearLayout>
            <ImageButton
                android:id="@+id/activity_more"
                android:layout_width="32dp"
                android:layout_height="32dp"
                android:layout_marginStart="8dp"
                android:padding="8dp"
                android:background="@drawable/user_detail_background"
                android:src="@drawable/ic_activity_more" />
        </LinearLayout>
        <com.google.android.material.chip.ChipGroup
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="14dp"
            app:chipSpacingHorizontal="8dp"
            app:chipSpacingVertical="8dp">
            <TextView
                android:id="@+id/activity_type"
                style="@style/purple_badge"
                android:text="@string/sample_activity_type" />
            <TextView
                android:id="@+id/activity_date"
                style="@style/UserDetail"
                android:drawableStart="@drawable/ic_user_calendar"
                android:text="@string/sample_activity_date" />
            <TextView
                android:id="@+id/activity_time"
                style="@style/UserDetail"
                android:drawableStart="@drawable/ic_activity_clock"
                android:text="@string/sample_activity_time" />
            <TextView
                android:id="@+id/activity_mode"
                style="@style/UserDetail"
                android:drawableStart="@drawable/ic_activity_people"
                android:text="@string/sample_activity_mode" />
        </com.google.android.material.chip.ChipGroup>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:orientation="horizontal"
            android:gravity="center_vertical">
            <TextView
                android:id="@+id/activity_organizer_avatar"
                android:layout_width="34dp"
                android:layout_height="34dp"
                android:background="@drawable/user_avatar_background"
                android:backgroundTint="#F0E8FF"
                android:gravity="center"
                android:text="@string/sample_activity_initials"
                android:textStyle="bold"
                android:textSize="12sp"
                android:textColor="#7944AD" />
            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginStart="8dp"
                android:layout_marginEnd="8dp"
                android:orientation="vertical">
                <TextView
                    android:id="@+id/activity_organizer_name"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@string/sample_activity_organizer"
                    android:textSize="12sp"
                    android:textStyle="bold"
                    android:textColor="#17263B" />
                <TextView
                    android:id="@+id/activity_organizer_role"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="2dp"
                    android:text="@string/sample_activity_role"
                    android:textSize="10sp"
                    android:textColor="#8A99AD" />
            </LinearLayout>
            <com.google.android.material.button.MaterialButton
                android:id="@+id/activity_secondary_action"
                style="@style/Widget.Material3.Button.OutlinedButton"
                android:layout_width="wrap_content"
                android:layout_height="36dp"
                android:minWidth="0dp"
                android:paddingHorizontal="10dp"
                android:insetTop="0dp"
                android:insetBottom="0dp"
                android:text="@string/sample_activity_secondary"
                android:textColor="#526375"
                android:textSize="11sp"
                android:textAllCaps="false"
                app:cornerRadius="100dp"
                app:strokeColor="#E7EBF1"
                app:strokeWidth="1dp"
                app:backgroundTint="#F7F9FB" />
            <com.google.android.material.button.MaterialButton
                android:id="@+id/activity_primary_action"
                android:layout_width="wrap_content"
                android:layout_height="36dp"
                android:layout_marginStart="8dp"
                android:minWidth="0dp"
                android:paddingHorizontal="12dp"
                android:insetTop="0dp"
                android:insetBottom="0dp"
                android:background="@drawable/activity_action_background"
                android:backgroundTint="@null"
                app:backgroundTint="@null"
                android:text="@string/sample_activity_primary"
                android:textColor="#FFFFFF"
                android:textSize="11sp"
                android:textStyle="bold"
                android:textAllCaps="false"
                app:icon="@drawable/ic_user_chevron"
                app:iconTint="#FFFFFF"
                app:iconSize="14dp"
                app:iconGravity="textEnd"
                app:iconPadding="6dp" />
        </LinearLayout>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

### `app/src/main/res/layout/activity_admin_menu_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#FFFFFF"
    android:paddingHorizontal="16dp">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="64dp"
        android:orientation="horizontal"
        android:gravity="center_vertical">
        <ImageButton
            android:id="@+id/menu_back"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:padding="8dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:src="@drawable/ic_list_back" />
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="@string/menu_title"
            android:textSize="22sp"
            android:textStyle="bold"
            android:textColor="#101A23" />
        <com.google.android.material.button.MaterialButton
            android:id="@+id/menu_add"
            android:layout_width="wrap_content"
            android:layout_height="44dp"
            android:minWidth="0dp"
            android:paddingHorizontal="14dp"
            android:insetTop="0dp"
            android:insetBottom="0dp"
            android:text="@string/menu_add"
            android:textSize="12sp"
            android:textColor="#FFFFFF"
            android:textAllCaps="false"
            app:icon="@drawable/ic_list_plus"
            app:iconSize="18dp"
            app:iconTint="#FFFFFF"
            app:backgroundTint="#08783F"
            app:cornerRadius="100dp" />
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="52dp"
        android:layout_marginTop="12dp"
        android:orientation="horizontal">
        <EditText
            android:id="@+id/menu_search"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@drawable/menu_search_background"
            android:paddingHorizontal="18dp"
            android:drawableStart="@drawable/ic_list_search"
            android:drawablePadding="12dp"
            android:hint="@string/menu_search"
            android:textSize="14sp"
            android:textColor="#14231D"
            android:textColorHint="#8B929E"
            android:inputType="text"
            android:singleLine="true" />
        <ImageButton
            android:id="@+id/menu_sort"
            android:layout_width="52dp"
            android:layout_height="52dp"
            android:layout_marginStart="10dp"
            android:padding="14dp"
            android:background="@drawable/menu_filter_background"
            android:src="@drawable/ic_list_filter"
            android:tint="#08783F" />
    </LinearLayout>
    <com.google.android.material.chip.ChipGroup
        android:id="@+id/menu_categories"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="8dp"
        app:singleSelection="true"
        app:selectionRequired="true"
        app:chipSpacingHorizontal="6dp"
        app:chipSpacingVertical="2dp">
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_all"
            style="@style/MenuFilterChip"
            android:text="@string/menu_all"
            app:chipIcon="@drawable/ic_list_grid"
            android:checked="true" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_food"
            style="@style/MenuFilterChip"
            android:text="@string/menu_food"
            app:chipIcon="@drawable/ic_item_food" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_drinks"
            style="@style/MenuFilterChip"
            android:text="@string/menu_drinks"
            app:chipIcon="@drawable/ic_list_drink" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_desserts"
            style="@style/MenuFilterChip"
            android:text="@string/menu_desserts"
            app:chipIcon="@drawable/ic_item_leaf" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_breakfast"
            style="@style/MenuFilterChip"
            android:text="@string/menu_breakfast"
            app:chipIcon="@drawable/ic_list_sun" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_lunch"
            style="@style/MenuFilterChip"
            android:text="@string/menu_lunch"
            app:chipIcon="@drawable/ic_list_lunch" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_dinner"
            style="@style/MenuFilterChip"
            android:text="@string/menu_dinner"
            app:chipIcon="@drawable/ic_list_moon" />
    </com.google.android.material.chip.ChipGroup>
    <TextView
        android:id="@+id/menu_empty"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="24dp"
        android:gravity="center"
        android:text="@string/menu_empty"
        android:textColor="#8B929E"
        android:visibility="gone" />
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/menu_list"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:clipToPadding="false"
        android:paddingBottom="16dp"
        tools:listitem="@layout/sample_item_card_list"
        tools:itemCount="3"
        app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
        app:spanCount="2" />
</LinearLayout>
```

### `app/src/main/res/drawable/ic_user_chevron.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M9,5 L16,12 L9,19" />
</vector>
```

### `app/src/main/res/drawable/item_action_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#40FFFFFF">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="315"
                android:startColor="#25CE99"
                android:endColor="#00A77B" />
        </shape>
    </item>
</ripple>
```

### `app/src/main/res/drawable/ic_item_star.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="14dp"
    android:height="14dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#FFBA24"
        android:strokeColor="#FFBA24"
        android:strokeWidth="0"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,1 L15.4,8.1 L23,9.2 L17.5,14.6 L18.8,22.2 L12,18.6 L5.2,22.2 L6.5,14.6 L1,9.2 L8.6,8.1 Z" />
</vector>
```

### `app/src/main/res/drawable/ic_item_leaf.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="14dp"
    android:height="14dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#10B981"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M4,19 C0,8 8,3 21,3 C21,16 15,23 4,19 Z M2,23 L21,3" />
</vector>
```

### `app/src/main/res/drawable/user_detail_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <solid
        android:color="#EFF4F6" />
    <corners
        android:radius="100dp" />
</shape>
```

### `app/src/main/res/drawable/ic_item_food.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="14dp"
    android:height="14dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#FF8A17"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,2 V8 Q3,11 7,11 Q11,11 11,8 V2 M7,2 V22 M16,22 V13 C13,12 14,2 18,2 C22,2 23,12 20,13 H16" />
</vector>
```

### `app/src/main/res/drawable/item_info_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <solid
        android:color="#FFF5EB" />
    <corners
        android:radius="100dp" />
</shape>
```

### `app/src/main/res/drawable/green_badge.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:paddingMode="stack">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#F1FCF8"
                android:endColor="#E6FAF2" />
            <stroke
                android:width="1dp"
                android:color="#D7F6EC" />
            <padding
                android:left="28dp"
                android:right="12dp"
                android:top="6dp"
                android:bottom="6dp" />
        </shape>
    </item>
    <item
        android:width="8dp"
        android:height="8dp"
        android:left="12dp"
        android:gravity="left|center_vertical">
        <shape
            android:shape="oval">
            <solid
                android:color="#10BE8B" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/drawable/ic_item_heart.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="22dp"
    android:height="22dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#121B2A"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,21 C8,17 2,13 2,8 C2,2 9,1 12,6 C15,1 22,2 22,8 C22,13 16,17 12,21 Z" />
</vector>
```

### `app/src/main/res/drawable/user_avatar_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <solid
        android:color="#E3F8F0" />
    <corners
        android:radius="100dp" />
</shape>
```

### `app/src/main/res/drawable/sample_item_photo.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="180dp"
    android:height="112dp"
    android:viewportWidth="180"
    android:viewportHeight="112">
    <path
        android:fillColor="#E6F7EE"
        android:pathData="M0,0H180V112H0Z" />
    <path
        android:fillColor="#8ECDB0"
        android:pathData="M10,102L60,42L98,84L124,56L170,102Z" />
    <path
        android:fillColor="#FFFFFF"
        android:pathData="M126,14a12,12 0,1 0,0,24a12,12 0,1 0,0,-24" />
</vector>
```

### `app/src/main/res/drawable/ic_list_moon.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M20,15a9,9 0,0 1,-11,-12a9,9 0,1 0,11,12" />
</vector>
```

### `app/src/main/res/color/menu_chip_text.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_checked="true"
        android:color="#08783F" />
    <item
        android:color="#18232F" />
</selector>
```

### `app/src/main/res/color/menu_chip_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_checked="true"
        android:color="#E6F7EE" />
    <item
        android:color="#F5F6F8" />
</selector>
```

### `app/src/main/res/drawable/ic_list_lunch.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,18H21 M5,16a7,7 0,0 1,14,0Z M12,7V4 M10,4H14 M2,21H22" />
</vector>
```

### `app/src/main/res/drawable/ic_list_sun.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,6a6,6 0,1 0,0,12a6,6 0,1 0,0,-12 M12,1V3 M12,21V23 M1,12H3 M21,12H23 M4,4L5.5,5.5 M18.5,18.5L20,20 M4,20L5.5,18.5 M18.5,5.5L20,4" />
</vector>
```

### `app/src/main/res/drawable/ic_list_drink.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M6,8H18L16,21H8Z M12,8L14,2H19 M6,12H18" />
</vector>
```

### `app/src/main/res/drawable/ic_list_grid.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,3H9V9H3Z M15,3H21V9H15Z M3,15H9V21H3Z M15,15H21V21H15Z" />
</vector>
```

### `app/src/main/res/drawable/ic_list_filter.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M4,6H8 M12,6H20 M4,12H14 M18,12H20 M4,18H6 M10,18H20 M8,6a2,2 0,1 0,4,0a2,2 0,1 0,-4,0 M14,12a2,2 0,1 0,4,0a2,2 0,1 0,-4,0 M6,18a2,2 0,1 0,4,0a2,2 0,1 0,-4,0" />
</vector>
```

### `app/src/main/res/drawable/menu_filter_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid
        android:color="#FFFFFF" />
    <stroke
        android:width="1dp"
        android:color="#EEF0F3" />
</shape>
```

### `app/src/main/res/drawable/ic_list_search.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M10.5,3.5a7,7 0,1 0,0,14a7,7 0,1 0,0,-14 M16,16L21,21" />
</vector>
```

### `app/src/main/res/drawable/menu_search_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="100dp" />
    <solid
        android:color="#F5F6F8" />
</shape>
```

### `app/src/main/res/drawable/ic_list_plus.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,4V20 M4,12H20" />
</vector>
```

### `app/src/main/res/drawable/ic_list_back.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#14231D"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M19,12H5 M12,5L5,12L12,19" />
</vector>
```

### `app/src/main/res/drawable/activity_action_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#40FFFFFF">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#9A58FF"
                android:endColor="#7225EC" />
        </shape>
    </item>
</ripple>
```

### `app/src/main/res/drawable/ic_activity_people.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M9,3 A4,4 0,1 1,9 11 A4,4 0,1 1,9 3 M2,21 V18 C2,12 16,12 16,18 V21 M17,3 C22,3 22,11 17,11 M19,14 C23,15 23,19 23,21" />
</vector>
```

### `app/src/main/res/drawable/ic_activity_clock.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,2 A10,10 0,1 1,12 22 A10,10 0,1 1,12 2 M12,6 V12 L16,14" />
</vector>
```

### `app/src/main/res/drawable/ic_user_calendar.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M5,5 H19 Q21,5 21,7 V20 Q21,22 19,22 H5 Q3,22 3,20 V7 Q3,5 5,5 Z M3,10 H21 M8,2 V7 M16,2 V7" />
</vector>
```

### `app/src/main/res/drawable/purple_badge.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:paddingMode="stack">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="#F8F2FF"
                android:endColor="#EEDFFF" />
            <stroke
                android:width="1dp"
                android:color="#E8D9FC" />
            <padding
                android:left="28dp"
                android:right="12dp"
                android:top="6dp"
                android:bottom="6dp" />
        </shape>
    </item>
    <item
        android:width="8dp"
        android:height="8dp"
        android:left="12dp"
        android:gravity="left|center_vertical">
        <shape
            android:shape="oval">
            <solid
                android:color="#9A62DE" />
        </shape>
    </item>
</layer-list>
```

### `app/src/main/res/drawable/ic_activity_more.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="#526572"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,3 A1,1 0,1 1,12 5 A1,1 0,1 1,12 3 M12,11 A1,1 0,1 1,12 13 A1,1 0,1 1,12 11 M12,19 A1,1 0,1 1,12 21 A1,1 0,1 1,12 19" />
</vector>
```

### `app/src/main/res/drawable/ic_user_email.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,5 H21 V19 H3 Z M3,5 L12,12 L21,5" />
</vector>
```

### `app/src/main/res/drawable/activity_icon_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="14dp" />
    <gradient
        android:angle="270"
        android:startColor="#F5EFFF"
        android:endColor="#EEE3FC" />
</shape>
```

### `app/src/main/res/drawable/activity_card_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="20dp" />
    <gradient
        android:angle="225"
        android:startColor="#FBF8FF"
        android:centerColor="#FFFFFF"
        android:endColor="#FFFFFF" />
</shape>
```

### `app/src/main/res/drawable/ic_user_certificate.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M2,8 L12,3 L22,8 L12,13 Z M6,10 V17 C9,20 15,20 18,17 V10 M22,8 V17" />
</vector>
```

### `app/src/main/res/drawable/ic_user_id.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,4 H21 V20 H3 Z M8,8 A2,2 0,1 1,8 12 A2,2 0,1 1,8 8 M5,17 C5,13 11,13 11,17 Z M14,8 H18 M14,12 H18 M14,16 H18" />
</vector>
```

### `app/src/main/res/drawable/ic_user_location.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,22 C9,18 4,13 4,9 A8,8 0,1 1,20 9 C20,13 15,18 12,22 Z M12,6 A3,3 0,1 1,12 12 A3,3 0,1 1,12 6" />
</vector>
```

### `app/src/main/res/drawable/ic_user_credits.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M20,6 C20,9 4,9 4,6 C4,3 20,3 20,6 Z M4,6 V11 C4,14 20,14 20,11 V6 M4,11 V16 C4,19 20,19 20,16 V11 M4,16 V20 C4,23 20,23 20,20 V16" />
</vector>
```

### `app/src/main/res/drawable/ic_user_phone.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#526572"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M6,3 L3,5 C3,13 11,21 19,21 L21,18 L16,14 L13,17 C10,15 8,13 7,10 L10,7 Z" />
</vector>
```

### `app/src/main/res/drawable/modal_card.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="24dp" />
    <gradient
        android:angle="225"
        android:startColor="@color/success_pale"
        android:centerColor="#F7FCFA"
        android:endColor="#FFFFFF" />
</shape>
```

### `app/src/main/res/values/list_template_resources.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color
        name="badge_green_text">#12805C</color>
    <color
        name="badge_purple_text">#7944AD</color>
    <color
        name="success_pale">#E3F8F0</color>
    <string
        name="item_available">Available</string>
    <string
        name="item_view_details">View Details</string>
    <string
        name="menu_add">Add Item</string>
    <string
        name="menu_all">All  3</string>
    <string
        name="menu_breakfast">Breakfast  1</string>
    <string
        name="menu_desserts">Desserts  0</string>
    <string
        name="menu_dinner">Dinner  1</string>
    <string
        name="menu_drinks">Drinks  0</string>
    <string
        name="menu_empty">No matching menu items</string>
    <string
        name="menu_food">Food  3</string>
    <string
        name="menu_lunch">Lunch  1</string>
    <string
        name="menu_search">Search menu items…</string>
    <string
        name="menu_title">Menu Items</string>
    <string
        name="sample_activity_date">Feb 5, 2024</string>
    <string
        name="sample_activity_initials">AR</string>
    <string
        name="sample_activity_mode">Online</string>
    <string
        name="sample_activity_organizer">Alex Rivera</string>
    <string
        name="sample_activity_primary">Accept</string>
    <string
        name="sample_activity_role">Organizer</string>
    <string
        name="sample_activity_secondary">Decline</string>
    <string
        name="sample_activity_subtitle">by Alex Rivera</string>
    <string
        name="sample_activity_time">6:00 PM – 7:00 PM</string>
    <string
        name="sample_activity_title">You’re invited to Web Development Class</string>
    <string
        name="sample_activity_type">Invitation</string>
    <string
        name="sample_item_category">Fast Food</string>
    <string
        name="sample_item_detail">320 calories</string>
    <string
        name="sample_item_location">Manila, PH</string>
    <string
        name="sample_item_name">Cheeseburger</string>
    <string
        name="sample_item_price">₱120</string>
    <string
        name="sample_item_rating">4.8 (120)</string>
    <string
        name="sample_item_tag">Beef</string>
    <string
        name="sample_user_age">29 years old</string>
    <string
        name="sample_user_credits">250 credits</string>
    <string
        name="sample_user_email">jamie.reyes@example.com</string>
    <string
        name="sample_user_id">ID: USR-00421</string>
    <string
        name="sample_user_initials">AM</string>
    <string
        name="sample_user_joined">Joined Jan 12, 2024</string>
    <string
        name="sample_user_location">Manila, PH</string>
    <string
        name="sample_user_name">Alex Morgan</string>
    <string
        name="sample_user_phone">0917 000 0001</string>
    <string
        name="sample_user_role">Member</string>
    <string
        name="sample_user_specialty">Certified Level 2</string>
    <string
        name="user_active">Active</string>
    <style
        name="ItemChip"
        parent="">
        <item
            name="android:layout_width">wrap_content</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:paddingHorizontal">6dp</item>
        <item
            name="android:paddingVertical">5dp</item>
        <item
            name="android:drawablePadding">4dp</item>
        <item
            name="android:gravity">center_vertical</item>
        <item
            name="android:textSize">10sp</item>
        <item
            name="android:includeFontPadding">false</item>
    </style>
    <style
        name="ItemChip.Info"
        parent="ItemChip">
        <item
            name="android:background">@drawable/item_info_background</item>
        <item
            name="android:textColor">#263449</item>
    </style>
    <style
        name="ItemChip.Tag"
        parent="ItemChip">
        <item
            name="android:background">@drawable/user_detail_background</item>
        <item
            name="android:backgroundTint">#E9FAF3</item>
        <item
            name="android:textColor">#138764</item>
    </style>
    <style
        name="ItemPhotoShape"
        parent="">
        <item
            name="cornerSize">12dp</item>
    </style>
    <style
        name="MenuFilterChip"
        parent="Widget.Material3.Chip.Filter">
        <item
            name="android:layout_width">wrap_content</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:textSize">12sp</item>
        <item
            name="android:textColor">@color/menu_chip_text</item>
        <item
            name="android:checkable">true</item>
        <item
            name="chipBackgroundColor">@color/menu_chip_background</item>
        <item
            name="chipStrokeWidth">0dp</item>
        <item
            name="chipCornerRadius">100dp</item>
        <item
            name="chipMinHeight">40dp</item>
        <item
            name="chipIconSize">20dp</item>
        <item
            name="chipIconTint">@color/menu_chip_text</item>
        <item
            name="checkedIconVisible">false</item>
        <item
            name="chipIconVisible">true</item>
        <item
            name="ensureMinTouchTargetSize">false</item>
    </style>
    <style
        name="StatusBadge"
        parent="">
        <item
            name="android:layout_width">wrap_content</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:minHeight">28dp</item>
        <item
            name="android:gravity">center_vertical</item>
        <item
            name="android:includeFontPadding">false</item>
        <item
            name="android:textSize">12sp</item>
        <item
            name="android:textStyle">bold</item>
    </style>
    <style
        name="UserAvatarShape"
        parent="">
        <item
            name="cornerSize">50%</item>
    </style>
    <style
        name="UserDetail"
        parent="">
        <item
            name="android:layout_width">wrap_content</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:background">@drawable/user_detail_background</item>
        <item
            name="android:paddingHorizontal">10dp</item>
        <item
            name="android:paddingVertical">7dp</item>
        <item
            name="android:drawablePadding">7dp</item>
        <item
            name="android:gravity">center_vertical</item>
        <item
            name="android:textColor">#344553</item>
        <item
            name="android:drawableTint">#526572</item>
        <item
            name="android:textSize">11sp</item>
        <item
            name="android:includeFontPadding">false</item>
    </style>
    <style
        name="UserDetail.Green"
        parent="UserDetail">
        <item
            name="android:backgroundTint">#E3F8F0</item>
        <item
            name="android:textColor">#08784F</item>
        <item
            name="android:drawableTint">#08784F</item>
    </style>
    <style
        name="green_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/green_badge</item>
        <item
            name="android:textColor">@color/badge_green_text</item>
    </style>
    <style
        name="purple_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/purple_badge</item>
        <item
            name="android:textColor">@color/badge_purple_text</item>
    </style>
</resources>
```

## 20. Generic dashboard template and complete implementation

Use this approved emerald/mint dashboard design for dashboard screens unless the user requests a different design. `DashboardSampleActivity` and `activity_dashboard_sample.xml` are the generic visual reference. Actual application dashboards remain separate per role, such as `AdminDashboardActivity`, `CustomerDashboardActivity`, and `InstructorDashboardActivity`, with separately owned layouts and behavior. Do not route all roles to the sample Activity or use it as a shared dashboard implementation.

Keep sample copy generic: Sample App, Dashboard Overview, SU, Welcome back!, Here’s an overview of your activity., and Users / Items / Requests / Completed. Do not put GymBuddy, gym, members, trainers, classes, enrollments, or another domain-specific brand into the generic template. For a real requested app, adapt the brand, identity and metrics to that app and role. Sample totals and growth percentages are visual placeholders, not database statistics; bind actual counts/calculated trends when that functionality is requested. Do not create tables or a backend just to reproduce this dashboard.

### Required visual structure

- White background, system sans-serif, 20dp horizontal padding, system-bar insets respected.
- A 52dp header with 16dp top margin: 44dp emerald rounded-square grid/logo icon, bold 20sp app name and gray 13sp dashboard subtitle, a 40dp notification button, and a 40dp mint circular initials avatar separated by 10dp. When the notification list has items, the bell button uses a pale mint background, green icon, and visible small red dot. When the list is empty, use a plain pale-gray background, gray icon, and hide the dot. Derive both states from the same notification list displayed in the bottom sheet; never leave the red dot visible for an empty list.
- Greeting 20dp below the header: bold 26sp #101A23. Description below it with 5dp margin, 13sp #8191A1.
- Four summary cards arranged in two equal-width columns and two rows, starting 20dp below the description. Cards are 152dp high, with 12dp column and row gaps, 18dp corners, no elevation, a 1dp #DDF5EA outline, and 14dp internal padding. When a summary represents a list the user needs to open, use THAT summary card as the navigation target: Users opens the role-specific user/student list, Items opens the role-specific menu/item list, and so on. Use a normal Intent, keep the destination role-specific, and make only cards with real destinations clickable. Do not add a separate menu of duplicate navigation buttons.
- Card background is #F7FDFB with subtle mint waves (#E9FAF2 and #DFF6EB) at the bottom. Draw the waves with the included native vector, never a downloaded image. Top row has a 40dp mint circular local icon panel and a small pale green trend pill on the right. Trend text is bold 12sp #08783F. The number is bold 28sp #101A23, with 12dp top spacing; label is 14sp #8191A1.
- Important List header is 24dp below the second card row. It has a 44dp mint icon panel on the left, bold 18sp title with gray 12sp subtitle, and a gray rounded count badge at the FAR RIGHT edge. Keep the text column weighted; the badge must never sit immediately beside the title. Badge minimum width is 32dp, with 10dp horizontal and 6dp vertical padding, bold 13sp #8191A1 text and #F5F6F8 background.
- Place a RecyclerView directly below the Important List header, with 12dp top margin, vertical LinearLayoutManager, remaining-height weight 1, and 16dp bottom padding. Do not hardcode repeated activity rows into the dashboard XML or wrap the RecyclerView in a ScrollView.
- For this sample, leave the RecyclerView EMPTY with no adapter, sample records or tools:listitem entries. Count is 0 because there are no displayed records. When a real activity feed is requested, bind appropriate ActivityCard rows and update the count from that same collection. Do not fabricate a nonzero count for an empty feed.
- Reuse shared XML drawables and installed offline-compatible AndroidX/Material components. Keep Java short, source comment-free, and every XML attribute on its own line. Do not add abstraction layers or extra dashboard helpers.

### Complete sample files

These files reproduce the dashboard without the original project or generated design image. Section 23 adds the approved notification bottom sheet, RecyclerView row and bell state resources. Replace `com.example.app` with the destination package. Shared resources `user_avatar_background.xml`, `user_detail_background.xml`, `menu_search_background.xml`, `ic_list_grid.xml`, `ic_activity_people.xml`, `ic_user_calendar.xml`, and `ic_user_id.xml` are already fully embedded in section 19; recreate those exact blocks if missing. Merge string/style entries by name and do not duplicate shared resources. The new dashboard-specific files are below.

### `app/src/main/java/com/example/app/DashboardSampleActivity.java`

```java
package com.example.app;

import android.os.Bundle;
import android.view.View;
import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import com.example.app.adapters.NotificationAdapter;
import com.example.app.models.NotificationItem;
import java.util.ArrayList;
import com.google.android.material.bottomsheet.BottomSheetDialog;

public class DashboardSampleActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_dashboard_sample);
        ArrayList<NotificationItem> notifications = sampleNotifications();
        boolean hasNotifications = !notifications.isEmpty();
        findViewById(R.id.dashboard_bell_icon).setSelected(hasNotifications);
        findViewById(R.id.dashboard_notification_dot).setVisibility(
                hasNotifications ? View.VISIBLE : View.GONE);
        findViewById(R.id.dashboard_notifications_button).setOnClickListener(v -> {
            BottomSheetDialog sheet = new BottomSheetDialog(this);
            sheet.setContentView(R.layout.bottom_sheet_notifications);
            RecyclerView list = sheet.findViewById(R.id.notifications_list);
            list.setLayoutManager(new LinearLayoutManager(this));
            list.setAdapter(new NotificationAdapter(this, notifications));
            sheet.findViewById(R.id.notifications_close).setOnClickListener(close -> sheet.dismiss());
            sheet.show();
        });
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (view, insets) -> {
            Insets bars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            int padding = Math.round(20 * getResources().getDisplayMetrics().density);
            view.setPadding(bars.left + padding, bars.top, bars.right + padding, bars.bottom);
            return insets;
        });
    }
    private ArrayList<NotificationItem> sampleNotifications() {
        ArrayList<NotificationItem> notifications = new ArrayList<>();
        String[] titles = {"New request received", "Item updated", "Task completed", "New message", "Reminder"};
        String[] messages = {"A new request is ready to review.", "An item was recently updated.",
                "A task was completed successfully.", "You received a new message.",
                "You have an upcoming activity."};
        String[] times = {"Just now", "5 min ago", "18 min ago", "1 hour ago", "Yesterday"};
        int[] icons = {R.drawable.ic_activity_people, R.drawable.ic_list_grid,
                R.drawable.ic_user_certificate, R.drawable.ic_user_email, R.drawable.ic_user_calendar};
        for (int i = 0; i < titles.length; i++) {
            NotificationItem item = new NotificationItem();
            item.title = titles[i];
            item.message = messages[i];
            item.time = times[i];
            item.icon = icons[i];
            notifications.add(item);
        }
        return notifications;
    }
}
```

### `app/src/main/res/layout/activity_dashboard_sample.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#FFFFFF"
    android:paddingHorizontal="20dp">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="52dp"
        android:layout_marginTop="16dp"
        android:orientation="horizontal"
        android:gravity="center_vertical">
        <ImageView
            android:layout_width="44dp"
            android:layout_height="44dp"
            android:padding="9dp"
            android:background="@drawable/dashboard_logo_background"
            android:src="@drawable/ic_list_grid"
            android:tint="#FFFFFF" />
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_marginStart="12dp"
            android:orientation="vertical">
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/dashboard_brand"
                android:textColor="#101A23"
                android:textSize="20sp"
                android:textStyle="bold" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/dashboard_panel"
                android:textColor="#8191A1"
                android:textSize="13sp" />
        </LinearLayout>
        <FrameLayout
            android:id="@+id/dashboard_notifications_button"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:foreground="?attr/selectableItemBackgroundBorderless">
            <ImageView
                android:id="@+id/dashboard_bell_icon"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:padding="9dp"
                android:background="@drawable/user_detail_background"
                android:backgroundTint="@color/dashboard_bell_background"
                android:src="@drawable/ic_dashboard_bell"
                android:tint="@color/dashboard_bell_tint" />
            <View
                android:id="@+id/dashboard_notification_dot"
                android:layout_width="8dp"
                android:layout_height="8dp"
                android:layout_gravity="top|end"
                android:layout_margin="5dp"
                android:background="@drawable/user_avatar_background"
                android:backgroundTint="#EF4260" />
        </FrameLayout>
        <TextView
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:layout_marginStart="10dp"
            android:background="@drawable/user_avatar_background"
            android:gravity="center"
            android:text="@string/dashboard_initials"
            android:textColor="#08783F"
            android:textStyle="bold"
            android:textSize="16sp" />
    </LinearLayout>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="@string/dashboard_greeting"
        android:textSize="26sp"
        android:textStyle="bold"
        android:textColor="#101A23" />
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="5dp"
        android:text="@string/dashboard_description"
        android:textSize="13sp"
        android:textColor="#8191A1" />
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:orientation="horizontal">
        <com.google.android.material.card.MaterialCardView
            style="@style/DashboardStatCard">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                android:padding="14dp"
                android:background="@drawable/dashboard_stat_background">
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="40dp"
                    android:orientation="horizontal"
                    android:gravity="center_vertical">
                    <ImageView
                        android:layout_width="40dp"
                        android:layout_height="40dp"
                        android:padding="9dp"
                        android:background="@drawable/user_avatar_background"
                        android:src="@drawable/ic_activity_people"
                        android:tint="#08783F" />
                    <Space
                        android:layout_width="0dp"
                        android:layout_height="1dp"
                        android:layout_weight="1" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:paddingHorizontal="10dp"
                        android:paddingVertical="5dp"
                        android:background="@drawable/menu_search_background"
                        android:backgroundTint="#E6F7EE"
                        android:text="@string/dashboard_users_growth"
                        android:textColor="#08783F"
                        android:textStyle="bold"
                        android:textSize="12sp" />
                </LinearLayout>
                <TextView
                    android:id="@+id/dashboard_users"
                    style="@style/DashboardStatNumber"
                    android:text="@string/dashboard_users_number" />
                <TextView
                    style="@style/DashboardStatLabel"
                    android:text="@string/dashboard_users" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>
        <com.google.android.material.card.MaterialCardView
            style="@style/DashboardStatCard"
            android:layout_marginStart="12dp">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                android:padding="14dp"
                android:background="@drawable/dashboard_stat_background">
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="40dp"
                    android:orientation="horizontal"
                    android:gravity="center_vertical">
                    <ImageView
                        android:layout_width="40dp"
                        android:layout_height="40dp"
                        android:padding="9dp"
                        android:background="@drawable/user_avatar_background"
                        android:src="@drawable/ic_list_grid"
                        android:tint="#08783F" />
                    <Space
                        android:layout_width="0dp"
                        android:layout_height="1dp"
                        android:layout_weight="1" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:paddingHorizontal="10dp"
                        android:paddingVertical="5dp"
                        android:background="@drawable/menu_search_background"
                        android:backgroundTint="#E6F7EE"
                        android:text="@string/dashboard_items_growth"
                        android:textColor="#08783F"
                        android:textStyle="bold"
                        android:textSize="12sp" />
                </LinearLayout>
                <TextView
                    android:id="@+id/dashboard_items"
                    style="@style/DashboardStatNumber"
                    android:text="@string/dashboard_items_number" />
                <TextView
                    style="@style/DashboardStatLabel"
                    android:text="@string/dashboard_items" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:orientation="horizontal">
        <com.google.android.material.card.MaterialCardView
            style="@style/DashboardStatCard">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                android:padding="14dp"
                android:background="@drawable/dashboard_stat_background">
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="40dp"
                    android:orientation="horizontal"
                    android:gravity="center_vertical">
                    <ImageView
                        android:layout_width="40dp"
                        android:layout_height="40dp"
                        android:padding="9dp"
                        android:background="@drawable/user_avatar_background"
                        android:src="@drawable/ic_user_calendar"
                        android:tint="#08783F" />
                    <Space
                        android:layout_width="0dp"
                        android:layout_height="1dp"
                        android:layout_weight="1" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:paddingHorizontal="10dp"
                        android:paddingVertical="5dp"
                        android:background="@drawable/menu_search_background"
                        android:backgroundTint="#E6F7EE"
                        android:text="@string/dashboard_requests_growth"
                        android:textColor="#08783F"
                        android:textStyle="bold"
                        android:textSize="12sp" />
                </LinearLayout>
                <TextView
                    android:id="@+id/dashboard_requests"
                    style="@style/DashboardStatNumber"
                    android:text="@string/dashboard_requests_number" />
                <TextView
                    style="@style/DashboardStatLabel"
                    android:text="@string/dashboard_requests" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>
        <com.google.android.material.card.MaterialCardView
            style="@style/DashboardStatCard"
            android:layout_marginStart="12dp">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                android:padding="14dp"
                android:background="@drawable/dashboard_stat_background">
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="40dp"
                    android:orientation="horizontal"
                    android:gravity="center_vertical">
                    <ImageView
                        android:layout_width="40dp"
                        android:layout_height="40dp"
                        android:padding="9dp"
                        android:background="@drawable/user_avatar_background"
                        android:src="@drawable/ic_user_id"
                        android:tint="#08783F" />
                    <Space
                        android:layout_width="0dp"
                        android:layout_height="1dp"
                        android:layout_weight="1" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:paddingHorizontal="10dp"
                        android:paddingVertical="5dp"
                        android:background="@drawable/menu_search_background"
                        android:backgroundTint="#E6F7EE"
                        android:text="@string/dashboard_completed_growth"
                        android:textColor="#08783F"
                        android:textStyle="bold"
                        android:textSize="12sp" />
                </LinearLayout>
                <TextView
                    android:id="@+id/dashboard_completed"
                    style="@style/DashboardStatNumber"
                    android:text="@string/dashboard_completed_number" />
                <TextView
                    style="@style/DashboardStatLabel"
                    android:text="@string/dashboard_completed" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:orientation="horizontal"
        android:gravity="center_vertical">
        <ImageView
            android:layout_width="44dp"
            android:layout_height="44dp"
            android:padding="10dp"
            android:background="@drawable/user_avatar_background"
            android:src="@drawable/ic_user_id"
            android:tint="#08783F" />
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_marginStart="12dp"
            android:layout_marginEnd="8dp"
            android:orientation="vertical">
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/dashboard_important"
                android:textSize="18sp"
                android:textStyle="bold"
                android:textColor="#101A23" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="3dp"
                android:text="@string/dashboard_important_subtitle"
                android:textSize="12sp"
                android:textColor="#8191A1" />
        </LinearLayout>
        <TextView
            android:id="@+id/dashboard_important_count"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:minWidth="32dp"
            android:paddingHorizontal="10dp"
            android:paddingVertical="6dp"
            android:gravity="center"
            android:background="@drawable/menu_search_background"
            android:text="@string/dashboard_count"
            android:textSize="13sp"
            android:textStyle="bold"
            android:textColor="#8191A1" />
    </LinearLayout>
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/dashboard_important_list"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:layout_marginTop="12dp"
        android:paddingBottom="16dp"
        android:clipToPadding="false"
        app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager" />
</LinearLayout>
```

### `app/src/main/res/values/dashboard_sample_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string
        name="dashboard_open">Open dashboard sample</string>
    <string
        name="dashboard_brand">Sample App</string>
    <string
        name="dashboard_panel">Dashboard Overview</string>
    <string
        name="dashboard_initials">SU</string>
    <string
        name="dashboard_greeting">Welcome back!</string>
    <string
        name="dashboard_description">Here’s an overview of your activity.</string>
    <string
        name="dashboard_important">Important List</string>
    <string
        name="dashboard_important_subtitle">Latest activities that need your attention.</string>
    <string
        name="dashboard_count">0</string>
    <string
        name="dashboard_users">Users</string>
    <string
        name="dashboard_items">Items</string>
    <string
        name="dashboard_requests">Requests</string>
    <string
        name="dashboard_completed">Completed</string>
    <string
        name="dashboard_users_number">248</string>
    <string
        name="dashboard_users_growth">↑ 12%</string>
    <string
        name="dashboard_items_number">36</string>
    <string
        name="dashboard_items_growth">↑ 8%</string>
    <string
        name="dashboard_requests_number">72</string>
    <string
        name="dashboard_requests_growth">↑ 5%</string>
    <string
        name="dashboard_completed_number">120</string>
    <string
        name="dashboard_completed_growth">↑ 18%</string>
    <string
        name="dashboard_notifications_title">Notifications</string>
    <string
        name="dashboard_notification_title">New update</string>
    <string
        name="dashboard_notification_message">You have a new activity to review.</string>
    <string
        name="dashboard_notification_time">Just now</string>
</resources>
```

### `app/src/main/res/values/dashboard_sample_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style
        name="DashboardStatCard"
        parent="Widget.Material3.CardView.Outlined">
        <item
            name="android:layout_width">0dp</item>
        <item
            name="android:layout_height">152dp</item>
        <item
            name="android:layout_weight">1</item>
        <item
            name="cardCornerRadius">18dp</item>
        <item
            name="cardElevation">0dp</item>
        <item
            name="strokeWidth">1dp</item>
        <item
            name="strokeColor">#DDF5EA</item>
        <item
            name="cardBackgroundColor">#F7FDFB</item>
    </style>
    <style
        name="DashboardStatNumber"
        parent="">
        <item
            name="android:layout_width">match_parent</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:layout_marginTop">12dp</item>
        <item
            name="android:textSize">28sp</item>
        <item
            name="android:textStyle">bold</item>
        <item
            name="android:textColor">#101A23</item>
    </style>
    <style
        name="DashboardStatLabel"
        parent="">
        <item
            name="android:layout_width">match_parent</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:textSize">14sp</item>
        <item
            name="android:textColor">#8191A1</item>
    </style>
</resources>
```

### `app/src/main/res/drawable/dashboard_stat_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="180dp"
    android:height="152dp"
    android:viewportWidth="180"
    android:viewportHeight="152">
    <path
        android:fillColor="#F7FDFB"
        android:pathData="M0,0H180V152H0Z" />
    <path
        android:fillColor="#E9FAF2"
        android:pathData="M65,152C90,115 112,142 136,100C151,76 168,72 180,72V152Z" />
    <path
        android:fillColor="#DFF6EB"
        android:pathData="M94,152C118,111 140,143 180,110V152Z" />
</vector>
```

### `app/src/main/res/drawable/dashboard_logo_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="14dp" />
    <gradient
        android:angle="315"
        android:startColor="#098B50"
        android:endColor="#05643B" />
</shape>
```

### `app/src/main/res/drawable/ic_dashboard_bell.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="#08783F"
        android:strokeWidth="2.2"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M5,17H19L17,14V9a5,5 0,0 0,-10,0V14Z M10,20a2,2 0,0 0,4,0 M12,2V4" />
</vector>
```

### Activity registration and sample navigation

Inside the existing manifest application element, register the internal sample screen:

```xml
<activity
    android:name=".DashboardSampleActivity"
    android:exported="false" />
```

To open the requested sample from an existing preview button, use the normal Activity Intent. Add only this button/listener to the existing screen; do not replace its unrelated content or create a new navigation framework:

```java
findViewById(R.id.open_dashboard_sample).setOnClickListener(v ->
        startActivity(new Intent(this, DashboardSampleActivity.class)));
```

The launching Activity imports `android.content.Intent`. The preview button XML uses the included dashboard_open string:

```xml
<com.google.android.material.button.MaterialButton
    android:id="@+id/open_dashboard_sample"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="24dp"
    android:text="@string/dashboard_open"
    android:textAllCaps="false"
    app:backgroundTint="#08783F" />
```

Declare `xmlns:app="http://schemas.android.com/apk/res-auto"` on the containing layout if missing. Build with `./gradlew --offline :app:assembleDebug`. Verify the cards fit, the count is at the far right and the empty RecyclerView occupies the space below the header. For a real app, summary cards may navigate to existing role-specific list Activities when those destinations are part of the requested workflow. Do not invent destinations solely for the sample dashboard.

## 21. Mandatory seeding and newest-first list behavior

When implementing a real Android app or machine problem, build the schema the requested workflows need in `DBHelper`. Seed every table in that schema with coherent local sample records. Seed data explicitly listed in the machine problem, and add sensible examples for the other required entities even when the problem gives none. For example, a food-ordering app needs several categories, foods, users/customers if used, orders, and order items; a pet adoption app needs species/breeds, pets, applicants, and applications if those entities are part of the requested flow. Give each list enough varied records to demonstrate scrolling, search, category/status filters, and the appropriate person/item/activity card. Use different statuses and realistic relationships. Keep all content offline, small, relevant, and free of real private information.

Add small `seed...` methods inside `DBHelper` or one simple `helpers/Seeder.java` if that is clearer for the problem. Create tables first and seed source rows before dependent rows so the plain ID values in records correspond to real sample data. Do not add SQLite foreign-key constraints. Run initial seeders from `onCreate(SQLiteDatabase db)` only after table creation. On a migration, seed newly added tables or missing defaults without deleting or duplicating existing records. Never reseed on every Activity launch or every list refresh. Do not wipe user edits with seed data. An idempotent seeder can check whether its table is empty before inserting:

```java
private void seedCategories(SQLiteDatabase db) {
    try (Cursor cursor = db.rawQuery("SELECT COUNT(*) FROM categories", null)) {
        cursor.moveToFirst();
        if (cursor.getInt(0) > 0) return;
    }
    for (String name : new String[]{"Food", "Drinks", "Desserts"}) {
        ContentValues values = new ContentValues();
        values.put("name", name);
        db.insertOrThrow("categories", null, values);
    }
}
```

This example assumes the real schema includes a `categories` table with a `name` column; replace those names with the actual model/table. Never paste the example into an unrelated project unchanged. Seed each actual table with the same simple pattern, including related child records, using the existing SQLite database handle passed to `onCreate`. If a table has a unique key, `insertWithOnConflict(..., SQLiteDatabase.CONFLICT_IGNORE)` may also make a migration seeder safe to rerun. Prefer a few useful records per table over large random datasets. Show seeded records in the correct approved RecyclerView row for their type.

Lists MUST show the latest created records first. `DBHelper.get()` in section 8 uses `id DESC` as the default for integer AUTOINCREMENT IDs. If the model has a reliable `createdAt` column, query with `createdAt DESC, id DESC` instead. Use a real creation time or increasing ID; do not sort by display name, status, or a mutable update time for the default latest view. Search and chip filtering retain that order. After a successful create, reload the list (or insert the new model at index 0) and update the adapter so the new row is visible at the top. Changing a filter or returning to the screen must not restore an old order. A user-selected alternative sort may temporarily change the view, but All/default returns to newest-first. When using in-memory preview records, add each new record at index 0 or sort by the same creation key.

Before finishing a list screen, check the actual rendered RecyclerView against the exact section 19 layout for its record type. Check first-row placement after creating a record, search/filter order, badge counts, chip icon meanings, and the modal/toast appearance after a transaction. A build alone does not confirm these visual and ordering requirements.

## 22. Simple ID relationships: never use SQLite foreign keys

This rule applies to every table and model in future projects. Never write `FOREIGN KEY (...)`, `REFERENCES other_table(id)`, `ON DELETE CASCADE`, or similar constraints. A relationship is only a normal integer value stored on the record: `userId`, `studentId`, `subjectId`, `itemId`, `orderId`, etc. In SQL use a plain column such as `studentId INTEGER`; in Java use `public int studentId;`. The column name must match the public field so DBHelper can map it directly. Do not create a `User user` or `Subject subject` field in a stored model or have DBHelper recursively load an object graph. This keeps DBHelper short and predictable.

If a detail screen needs another record's name, call `db.show(User.class, model.userId)` or the equivalent explicitly in that screen's loading method. If an ID is invalid or the related record was deleted, handle the missing result in the UI. Validate an ID when a transaction requires the related record to exist. Seed records in an order that makes their stored IDs meaningful, but do not add database foreign-key enforcement. Do not include foreign-key constraints even if the machine problem describes related entities; represent those relationships with the plain `...Id` fields above.

## 23. Dashboard notification drawer and conditional bell

The dashboard bell opens a bottom `BottomSheetDialog` using the locally available Material Components dependency. Use `bottom_sheet_notifications.xml` for the sheet, with a drag handle, Notifications title, close button and RecyclerView. The five visible entries come from a sample `ArrayList<NotificationItem>` bound through `NotificationAdapter(Context context, ArrayList<NotificationItem> list)`. The sheet XML contains no repeated hardcoded rows. Replace sample items with actual notifications when the application has them. Keep newest notifications first. The row is compact, mint/white, and uses a relevant local vector icon, title, message and relative time. Allow scrolling when more entries exist or the device is short.

The bell's visual state is conditional. Use the SAME list that feeds the RecyclerView: nonempty means pale mint `#E6F7EE` background, emerald `#08783F` bell and red dot; empty means pale gray `#F5F6F8` background, gray `#526375` bell and no dot. Refresh this state when notifications are added, read, or removed. A red dot with zero notifications is incorrect. The sample has five items, so its bell is colored. Do not create network push or system notifications unless requested.

Dashboard overview cards handle needed navigation. For example, a Users/Students summary card opens the matching role-specific Users/Students list; an Items/Menu summary card opens the matching role-specific Menu list. Make the whole card clickable and give it the normal ripple. Wire the Intent only when the destination Activity exists and the role is allowed to open it. Keep dashboard layouts/Activities separate per role, and do not add duplicate navigation buttons beneath the cards.

For an existing admin menu list, give the Items summary card the ID `dashboard_items_card` and attach its click in that role's dashboard Activity:

```java
findViewById(R.id.dashboard_items_card).setOnClickListener(v ->
        startActivity(new Intent(this, AdminMenuListActivity.class)));
```

Use the equivalent role-specific destination for other cards. Do not attach a listener to a card without a real destination. In XML, set `app:rippleColor="#2608783F"` on clickable MaterialCardView cards while preserving the approved card size and colors.

The current dashboard code, layout and strings in section 20 already include the notification connection and bell state. Add these supporting files when recreating the sample:

### `app/src/main/java/com/example/app/models/NotificationItem.java`

```java
package com.example.app.models;

public class NotificationItem {
    public String title, message, time;
    public int icon;
}
```

### `app/src/main/java/com/example/app/adapters/NotificationAdapter.java`

```java
package com.example.app.adapters;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.example.app.R;
import com.example.app.models.NotificationItem;
import java.util.ArrayList;

public class NotificationAdapter extends RecyclerView.Adapter<NotificationAdapter.NotificationViewHolder> {
    public Context context;
    public ArrayList<NotificationItem> list;

    public NotificationAdapter(Context context, ArrayList<NotificationItem> list) {
        this.context = context;
        this.list = list;
    }

    @NonNull
    @Override
    public NotificationViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        return new NotificationViewHolder(LayoutInflater.from(context)
                .inflate(R.layout.sample_notification_list, parent, false));
    }

    @Override
    public void onBindViewHolder(@NonNull NotificationViewHolder holder, int position) {
        NotificationItem item = list.get(position);
        holder.title.setText(item.title);
        holder.message.setText(item.message);
        holder.time.setText(item.time);
        holder.icon.setImageResource(item.icon);
    }

    @Override
    public int getItemCount() {
        return list.size();
    }

    public static class NotificationViewHolder extends RecyclerView.ViewHolder {
        ImageView icon;
        TextView title, message, time;

        public NotificationViewHolder(@NonNull View view) {
            super(view);
            icon = view.findViewById(R.id.notification_icon);
            title = view.findViewById(R.id.notification_title);
            message = view.findViewById(R.id.notification_message);
            time = view.findViewById(R.id.notification_time);
        }
    }
}
```

### `app/src/main/res/layout/bottom_sheet_notifications.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingHorizontal="20dp"
    android:paddingBottom="28dp"
    android:background="@drawable/dashboard_sheet_background">
    <View
        android:layout_width="36dp"
        android:layout_height="4dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"
        android:background="@drawable/menu_search_background"
        android:backgroundTint="#DDE5E1" />
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="22dp"
        android:orientation="horizontal"
        android:gravity="center_vertical">
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="@string/dashboard_notifications_title"
            android:textSize="22sp"
            android:textStyle="bold"
            android:textColor="#101A23" />
        <TextView
            android:id="@+id/notifications_close"
            android:layout_width="36dp"
            android:layout_height="36dp"
            android:background="@drawable/user_detail_background"
            android:gravity="center"
            android:text="×"
            android:textSize="24sp"
            android:textColor="#526375" />
    </LinearLayout>
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/notifications_list"
        android:layout_width="match_parent"
        android:layout_height="440dp"
        android:layout_marginTop="16dp"
        android:clipToPadding="false"
        android:paddingBottom="8dp" />
</LinearLayout>
```

### `app/src/main/res/layout/sample_notification_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginBottom="5dp"
    android:paddingHorizontal="12dp"
    android:paddingVertical="6dp"
    android:orientation="horizontal"
    android:gravity="center_vertical"
    android:background="@drawable/modal_card">
    <ImageView
        android:id="@+id/notification_icon"
        android:layout_width="42dp"
        android:layout_height="42dp"
        android:padding="10dp"
        android:background="@drawable/user_avatar_background"
        android:src="@drawable/ic_dashboard_bell"
        android:tint="#08783F" />
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:layout_marginStart="12dp"
        android:orientation="vertical">
        <TextView
            android:id="@+id/notification_title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/dashboard_notification_title"
            android:textSize="14sp"
            android:textStyle="bold"
            android:textColor="#101A23" />
        <TextView
            android:id="@+id/notification_message"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="2dp"
            android:text="@string/dashboard_notification_message"
            android:textSize="12sp"
            android:textColor="#8191A1" />
        <TextView
            android:id="@+id/notification_time"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:text="@string/dashboard_notification_time"
            android:textSize="10sp"
            android:textColor="#08783F" />
    </LinearLayout>
</LinearLayout>
```

### `app/src/main/res/drawable/dashboard_sheet_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:topLeftRadius="24dp"
        android:topRightRadius="24dp" />
    <solid
        android:color="#FFFFFF" />
</shape>
```

### `app/src/main/res/color/dashboard_bell_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_selected="true"
        android:color="#E6F7EE" />
    <item
        android:color="#F5F6F8" />
</selector>
```

### `app/src/main/res/color/dashboard_bell_tint.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_selected="true"
        android:color="#08783F" />
    <item
        android:color="#526375" />
</selector>
```

