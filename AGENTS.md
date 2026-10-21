# RestaurantApp UI contract — 2026-10-17

This file is the current source of truth for the app's reusable visual design. Follow the prose rules first, then use the complete XML resources in the appendix as the structural visual reference for the requested screen. Adapt domain text, IDs, and data bindings while preserving the reference's visible hierarchy, spacing, alignment, and shapes. The appendix is a historical snapshot of the XML files under `app/src/main/res` plus `AndroidManifest.xml`; newer prose rules and current live resources override stale colors, theme assignments, `@string` references, and separate style files shown there. In this repository, edit live files under `app/src/main/res` when implementation work is requested; do not create duplicate resources from the appendix. For another project, bring over the needed visual values and binary assets, consolidate styles into one `styles.xml`, and put static TextView copy inline as specified below. If a screenshot or older guide disagrees with a live resource here, the latest user request and live resource win. A destination project's existing appearance does not silently override this visual contract; preserve a different theme only when the user explicitly requests it.

## Global implementation rules

- Use Java and Android XML, ViewBinding (`buildFeatures { viewBinding = true }`), `AppCompatButton` for text buttons, and existing local dependencies. Keep code short and readable.
- Font: `app/src/main/res/font/arial_rounded_mt_bold.ttf`. Both `android:fontFamily` and `fontFamily` are set in `values/themes.xml`. Keep explicit font attributes where present. The TTF is a binary asset and must be copied with the project; it cannot be represented by XML. Do not assume a theme font affects every custom-drawn text or every widget automatically. The current `values-night/themes.xml` snapshot contains a sparse override without the explicit font and color items. When reproducing the design, copy the day theme's font and palette items into the night override too, or use one shared base theme that resolves identically in both modes; otherwise night mode can look different.
- Palette: authentication and role-neutral sample screens use the green `primary_darkest`, `primary_dark`, `primary`, `primary_lime`, and `primary_light` values. Every signed-in role screen uses the role palette system below: black and white for Admin, blue for Personal/member/customer/student/patient screens. Never hardcode the green primary resources in a role-owned or shared signed-in screen. Keep `success`, `destructive`, and other semantic status colors for actual success, danger, warning, availability, and workflow states. Keep `text_heading`, `text_body`, and `text_subheading` for readable neutral text. Drawables and styles must use theme attributes for role-colored surfaces rather than unrelated hex values or direct role color resources.
- Keep modal and top-toast appearance exactly as `layout/modal_status.xml`, `layout/toast_banner.xml`, their drawable/style resources, and `AppModal`/`AppToast`. State-changing actions use the confirmation/danger modal before the change and the toast after the actual result.
- Use the shared animated empty state `layout/view_empty_list.xml` for empty and zero-result lists. Seed each actual database table with useful preview data, including supporting tables, even when a machine problem omits example rows. Keep tables independent: no SQL foreign keys, joins, reference columns, or relationship lookups. Default lists show latest-created records first and retain that order through search/filter.

For an exact day/night transfer, replace the sparse night-theme snapshot in the appendix with the same font and palette items as the day theme. This is the intended `values-night/themes.xml` content for a destination project; the appendix below still records this project's current file verbatim:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style
        name="Base.Theme.RestaurantApp"
        parent="Theme.Material3.DayNight.NoActionBar">
        <item name="android:fontFamily">@font/arial_rounded_mt_bold</item>
        <item name="fontFamily">@font/arial_rounded_mt_bold</item>
        <item name="colorPrimary">@color/primary_dark</item>
        <item name="colorOnPrimary">@color/white</item>
        <item name="colorSecondary">@color/primary</item>
        <item name="colorOnSecondary">@color/text_heading</item>
        <item name="colorSurface">@color/white</item>
        <item name="colorOnSurface">@color/text_heading</item>
        <item name="colorError">@color/destructive</item>
        <item name="colorOnError">@color/white</item>
        <item name="android:colorAccent">@color/primary</item>
        <item name="rolePrimary">@color/primary</item>
        <item name="rolePrimaryDark">@color/primary_dark</item>
        <item name="rolePrimaryLight">@color/primary_light</item>
        <item name="roleAccent">@color/primary_lime</item>
        <item name="roleOnPrimary">@color/white</item>
        <item name="rolePageBackground">@color/white</item>
        <item name="roleOverlayStart">@color/black_75</item>
        <item name="roleOverlayEnd">@color/black_45</item>
    </style>
</resources>
```

## Mandatory role palette system

Keep the shared role attributes in `values/attrs.xml`: `rolePrimary`, `rolePrimaryDark`, `rolePrimaryLight`, `roleAccent`, `roleOnPrimary`, `rolePageBackground`, `roleOverlayStart`, and `roleOverlayEnd`. Define their green defaults in `Base.Theme.RestaurantApp`, then override them in `Theme.RestaurantApp.Admin` and `Theme.RestaurantApp.Personal`. Keep the same base attributes available in night mode so role screens do not fall back to unrelated platform colors.

Use these exact Admin colors:

| Color | Value |
| --- | --- |
| `admin_primary` | `#18181B` |
| `admin_primary_dark` | `#000000` |
| `admin_primary_light` | `#F1F1F2` |
| `admin_accent` | `#71717A` |
| `admin_page_background` | `#F7F7F8` |
| `admin_overlay_start` | `#E6000000` |
| `admin_overlay_end` | `#99000000` |

Use these exact Personal colors for every non-admin signed-in role such as member, customer, student, or patient:

| Color | Value |
| --- | --- |
| `personal_primary` | `#3B82F6` |
| `personal_primary_dark` | `#1D4ED8` |
| `personal_primary_light` | `#DBEAFE` |
| `personal_accent` | `#60A5FA` |
| `personal_page_background` | `#F5F8FF` |
| `personal_overlay_start` | `#D9172554` |
| `personal_overlay_end` | `#993B82F6` |

Assign `@style/Theme.RestaurantApp.Admin` in the manifest to every admin dashboard, list, create, update, show, profile, and report Activity. Assign `@style/Theme.RestaurantApp.Personal` to every personal dashboard, list, create, update, show, and profile Activity. A shared signed-in Activity such as `SampleActivityShowActivity` must call `setTheme(...)` **before** `super.onCreate(...)`, choosing Admin only when `StaticUserModel.user.role` equals `Admin` and Personal otherwise. An admin-only report or create page uses the Admin theme directly in the manifest. Login and registration stay on the default green `Theme.RestaurantApp` because no role has been selected yet.

Role colors must visibly apply to buttons, action gradients, dashboard feature and overview boxes, count badges, chips, avatars, icon circles, card accents, page backgrounds, sticky-header overlays, profile covers and controls, reports, activity/show pages, notifications, empty states, and confirmation modals. Shared XML uses `?attr/rolePrimary`, `?attr/rolePrimaryDark`, `?attr/rolePrimaryLight`, `?attr/roleAccent`, `?attr/roleOnPrimary`, `?attr/rolePageBackground`, `?attr/roleOverlayStart`, and `?attr/roleOverlayEnd`. Do not use `@color/primary`, `@color/primary_dark`, `@color/primary_light`, or `@color/primary_lime` inside signed-in role UI. Resolve the same attributes from Java with the Activity theme when runtime drawing or spans need a color; do not reference `R.color.primary_*` there.

Semantic colors remain independent of role. Use `success`/`badge_green` only for a true successful or available state, `destructive`/`badge_red` only for danger, deletion, cancellation, or failure, and `badge_yellow` only for a genuine warning. Do not use semantic green as a decorative endpoint of an Admin or Personal button gradient. Neutral white cards, field backgrounds, borders, and readable text colors can stay shared when they do not represent role identity.

## Mandatory Java package structure

Create these packages as soon as their first class is needed. Do not put all classes in the base package merely because Android Studio created it there.

| Package | What belongs there |
| --- | --- |
| base package | `SampleLoginActivity`, `SampleRegisterActivity`, `PersonalDashboardActivity`, and role-neutral sample/demo Activities; there is no `MainActivity` |
| `admin` (or the actual role name) | Every Activity owned by that role, such as `AdminDashboardActivity`, `AdminCarListActivity`, `AdminCarShowActivity`, and create/edit/report Activities |
| `models` | One plain model for each stored or displayed entity, including notification items |
| `adapters` | Every RecyclerView adapter and its ViewHolder; item/person/activity/notification lists must be bound through adapters rather than views built inside an Activity |
| `helpers` | `DBHelper`, `AppModal`, `AppToast`, seeders, PDF export, and shared business rules/utilities |

For example, a car-rental app should not leave `AdminCarListActivity`, `AppModal`, `Database`, `RentalRules`, and `ReportPdf` side by side in `com.example.carrental`. Put the admin screens in `com.example.carrental.admin`; models in `.models`; list adapters in `.adapters`; and `DBHelper`, modal/toast, rental rules, and PDF helper in `.helpers`. Use `DBHelper` as the SQLite helper name. Keep data fields in models, SQL in `DBHelper`, row binding in adapters, and screen setup/navigation in Activities. Update package declarations, imports, manifest Activity names, and affected tests whenever moving an existing class. Before finishing, inspect the Java tree and verify every class is in its designated package. Only omit a package when the feature truly has no class of that kind.

## Offline, code, and screen ownership rules

- Work with the Android Studio SDK, assets, Gradle plugins, and dependencies already available locally. AndroidX and Material widgets are allowed when already installed. Do not add a dependency, font, icon, image, HTTP backend, cloud database, remote authentication, or Internet permission that makes the app require a network connection.
- This is a hackathon app: prioritize working screens and exact design with the shortest clear Java/XML implementation. Avoid unnecessary architecture layers, repositories, dependency injection, generic frameworks, and extra files. Do not add try/catch for routine code; use it only when an API requires it or a real failure must be shown. Do not compress unrelated statements to save lines. Do not add Java or XML comments, Javadocs, TODOs, or commented-out code.
- Put every XML attribute on a separate line with four-space indentation in new or edited XML. Keep existing resource names and remove only resources genuinely made unused.
- For new or edited screens, put fixed visible copy directly in the view's XML attribute, for example `android:text="Activities"` on a `TextView`. Put static hints directly on the field too. Do not add screen copy to `strings.xml` or create feature-specific string resource files. Set runtime values from Java. Keep string resources only where Android requires a resource, such as the application label, or where a reusable system-facing value genuinely needs one.
- Put all reusable UI `<style>` definitions in one `app/src/main/res/values/styles.xml`. Do not create per-screen or per-component style files. When adapting a style from the appendix, move its definition into that one file and keep the style name and appearance. `themes.xml`, `values-night/themes.xml`, `colors.xml`, and `typography.xml` keep their separate platform/theme/token roles; do not merge them into `styles.xml`.
- Use `AppCompatButton` for text buttons and `AppCompatImageButton` for icon-only buttons. Preserve the exact existing modal, toast, badge, and card appearance. Never substitute stock `Toast`, `Snackbar`, or `AlertDialog` for the approved reusable modal/toast.
- Every role-owned Activity follows `<Role><EntityOrAction>Activity` and lives in its role package. Each role gets its own dashboard, list, create, update, show, and report Activity/layout when that screen is requested. Create and update are separate Activities and layouts, even if they share fields. Register each created Activity in the manifest and update references when moving it. Shared models, small components, drawables, and helpers remain reusable.
- RecyclerView lists require a model-specific row XML, adapter, and ViewHolder. Adapter constructors take only `Context` and the model list. Keep row binding and row-specific actions in the adapter; do not inflate repeated rows directly inside an Activity or pass extra callbacks as constructor/setter workarounds. Reset all variable visibility, image, badge, and text state during each bind. Never show passwords or private credentials in a row.
- Keep SQL in `helpers.DBHelper`. Models use plain public scalar fields. Tables are independent: do not add SQL joins, references, foreign keys, relationship IDs, or nested model graphs. Seed every table once with relevant sample data, including supporting entities; do not reseed on each launch or erase user edits. Use a reliable creation key for newest-first lists.
- A transaction changes data only after the appropriate confirmation/danger modal; Cancel changes nothing. Show the success/error toast after the real result, close cursors and streams, handle actual errors, and refresh the affected RecyclerView.
- For implementation changes, run the relevant offline build and inspect meaningful UI/transaction flows on an available emulator when practical, unless the user explicitly says not to test. Do not add tests that merely mirror a trivial layout. Documentation-only changes do not require an app build. Report what was and was not verified.

## Mandatory DBHelper format

Put one `DBHelper extends SQLiteOpenHelper` in `helpers`. Every persisted model in `models` declares a public static `TABLE_NAME`, a public no-argument constructor, and public scalar fields whose names match SQL columns. Map only `String`, `int`/`Integer`, and `double`/`Double`; represent flags as `int` 0/1. SQL columns use only `TEXT`, `INTEGER`, or `DOUBLE`; every non-ID column is nullable. Do not add `NOT NULL`, `COLLATE`, SQL joins, references, foreign keys, or nested model graphs. This hackathon app stores passwords as plain text; no hashing is required. Never display passwords in rows.

The helper must provide the same reflection-based CRUD API for every model, using `TABLE_NAME` and public fields rather than a separate row mapper or `ContentValues` builder for each entity:

```java
boolean create(Object model) throws ReflectiveOperationException;
boolean update(Object model, int id) throws ReflectiveOperationException;
boolean delete(Class<?> modelClass, int id) throws ReflectiveOperationException;
<T> T show(Class<T> modelClass, int id) throws ReflectiveOperationException;
<T> ArrayList<T> get(Class<T> modelClass) throws ReflectiveOperationException;
```

Implement this API with `getTableName`, `isColumnType`, `toContentValues`, and `fromCursor` reflection helpers. Skip static fields and `id` on writes; read `id` on queries. Use parameterized `id=?` selections, close every cursor, return `null` when `show` finds no row, and order `get` by `id DESC`. A custom column alias belongs in one shared mapping helper only when the schema genuinely requires it. Do not silently skip unsupported public model fields; either use the supported scalar type or deliberately extend the mapper for a real requirement. Keep routine database operations short; do not add defensive try/catch wrappers. Show a success toast only when the write succeeded.

Keep actual tables, seed data, and non-destructive migrations in `onCreate`/`onUpgrade`, preserving the destination project's database name and version. Domain methods may enforce business rules, authentication, or filtered queries, but should delegate ordinary model writes and reads to the generic CRUD methods. Do not replace this helper with per-model `car(...)`, `user(...)`, or similar mappers, Room, a repository layer, or an ORM.

## Fixed typography sizes

`app/src/main/res/values/typography.xml` is the single source for text sizes. New and edited XML must use these `@dimen/type_...` resources in `android:textSize` and styles; do not write raw `sp` values or choose sizes by eye. Keep the Arial Rounded font from the theme. Use the standard roles below for new screens:

| Role | Resource | Size | Typical use |
| --- | --- | --- | --- |
| Page heading | `@dimen/type_page_heading` | 22sp | List/create/show page title, sheet heading |
| Section heading | `@dimen/type_section_heading` | 20sp | Dashboard and detail section titles |
| Card heading | `@dimen/type_card_heading` | 18sp | Person, item, or activity card title |
| Body | `@dimen/type_body` | 14sp | Descriptions, search input, normal content |
| Subheading | `@dimen/type_subheading` | 13sp | Supporting line below a heading, metadata |
| Label | `@dimen/type_label` | 12sp | Field label, chip, badge, compact button label |
| Detail | `@dimen/type_detail` | 11sp | Dense card facts |
| Caption | `@dimen/type_caption` | 10sp | Timestamps and low-emphasis hints |
| Display | `@dimen/type_display` | 28sp | Large dashboard greeting or count |

The same file also defines fixed exception tokens for existing components: `type_micro` (9sp), `type_button` (15sp), `type_body_large` (16sp), `type_person_name` (17sp), `type_card_heading_large` (19sp), `type_report_section` (21sp), `type_large_heading` (24sp), `type_report_title` (25sp), `type_metric` (26sp), `type_dashboard_greeting` (27sp), `type_modal_symbol` (36sp), `type_report_total` (46sp), and `type_auth_hero` (34sp) for the login/registration hero heading. These preserve the approved current XML exactly; reuse them only for the matching component. Do not resize the modal, toast, report total, or existing cards just to reduce the token count. When changing a text size intentionally, change the shared token if all uses should change; otherwise choose an existing role token or add a clearly named component token and document it here.

## Login and registration pages

Use **two separate Activities and two separate layouts**: `SampleLoginActivity` with `layout/activity_sample_login.xml`, and `SampleRegisterActivity` with `layout/activity_sample_register.xml`. Role-specific equivalents also stay separate. Never use one Activity that changes modes to serve both screens, and never hide both forms inside one layout. Each link starts the other Activity; register both Activities in the manifest. `helpers/AuthPasswordVisibility.java` handles the login and registration password eye, while registration uses `ActivityResultContracts.GetContent()` to choose an image. Login is the launcher; remove `MainActivity`. Login checks local `DBHelper` users, assigns the signed-in user to `StaticUserModel.user`, then routes with a simple role check: `if ("Admin".equalsIgnoreCase(user.role))` open `AdminDashboardActivity`, otherwise open `PersonalDashboardActivity`. Registration creates a local user with role `Member` and opens the personal dashboard. `StaticUserModel` holds only the current in-memory session. Logout clears it and starts `SampleLoginActivity` directly, with no confirmation modal. No password hashing is needed for this hackathon.

Copy the **complete XML resources below** for the exact current appearance. Both pages use a 390dp hero with `auth_hero_gradient.xml`, which blends `primary_darkest` through `primary_dark` into `primary`. There is no scenic photo. Hero title text is white at `@dimen/type_auth_hero` (34sp), subtitle is `primary_light` at `type_body_large`, and the white back icon sits on the translucent `auth_back_background.xml`. The white `auth_sheet_background.xml` has 34dp rounded top corners and overlaps the hero by 34dp. The form uses labels **above every field**, 58dp pill inputs with `auth_field_background.xml` and the existing neutral field colors, inline email/lock icons, and a 60dp dark-primary pill action with `ic_auth_arrow.xml`. Use the exact spacing and ordering in each layout; preserve the Arial Rounded font and typography resources. Login includes email, password, forgot-password link, Sign In action, and Sign Up link. Registration includes a labeled profile-photo upload area using `form_upload_background.xml`, contained `fitCenter` preview, first/last name, email, password, confirmation, Create Account action, and Sign In link. Do not add the “Or continue with” section or Google/Facebook buttons. Keep the shared drawable colors resource-based so changes to `values/colors.xml` update both pages.

## Dashboard

Use `layout/activity_admin_dashboard.xml` as the admin sample. Keep its greeting and notification bell in the fixed header above the scrollable content. The dark feature panel is a `ViewFlipper` with three full-card slides: Update, Items, and Activities. Each child keeps the 166dp rounded card, its own primary/badge background shade, inline fixed label/date/action text, and the highlighted dot and figure. Use `autoStart="false"`, `flipInterval="3900"`, and the platform fade-in/fade-out animations. Bind only the live item and activity counts from Java, start flipping in `onStart`, and stop in `onStop`; do not reintroduce a rotating `Runnable` or manually swap the same TextViews. Tapping the card opens Reports. The two overview metric boxes navigate to the existing item and person lists; the left box has the floating shadow treatment. Put the item-card RecyclerView first and the activity-card RecyclerView second. The **admin** item cards use a two-column grid in this RestaurantApp reference, while activity cards use a vertical list. The personal dashboard has a separate mandatory single-row horizontal featured list below. Both admin sections have headings, subheadings, and right-side count badges; the activity section has its own search and wrapping status chips. A destination project can explicitly require a different item-list orientation; preserve that project's latest user instruction while keeping the shared card styling. Use the XML's actual background drawables, color selectors, and styles rather than rebuilding approximations. `DashboardSampleActivity` is the older generic sample; the admin dashboard is the current visual reference.

The admin sticky header is an image-backed `FrameLayout` with a fixed height of **150dp**. Keep it above and outside `admin_scroll`. Its original `dashboard_header_background.xml` remains on the `FrameLayout` to provide the rounded bottom outline; set `clipToOutline="true"` and `outlineProvider="background"`. Inside it, place a `match_parent` `ImageView` using `personal_seats_5` and `centerCrop`, then a full-size `View` using `dashboard_header_image_overlay.xml`, then the padded greeting/bell/avatar content as the final child. Use white heading and bell colors and `?attr/rolePrimaryLight` for the supporting line. The overlay uses `?attr/roleOverlayStart` and `?attr/roleOverlayEnd`, giving Admin its black overlay. Never apply a bitmap or bitmap `layer-list` directly as the background of a `wrap_content` header: the bitmap's intrinsic height can expand the header over the entire screen. Never place the header image after the content, because it would cover the controls.

The initials avatar (`admin_avatar_button`) beside the notification bell opens the anchored `popup_admin_account.xml` menu. Keep its 188dp white rounded panel, subtle outline and elevation, text-only Profile and destructive-colored Logout buttons, and separator. Use `admin_account_menu_background.xml` with inline button text rather than a platform-default menu. Profile opens the separate role-owned `admin.AdminProfileActivity` and `activity_admin_profile.xml`, using the shared profile design described below. Register this Activity in the manifest. Logout clears `StaticUserModel.user` and starts `SampleLoginActivity` directly; do not show a confirmation modal. Do not put the account menu inside the notification drawer or make the avatar tap log out directly.

## Personal dashboard and reusable CTA carousel

Use `PersonalDashboardActivity` and `activity_personal_dashboard.xml` for the role-neutral sample; a real student, customer, patient, or member screen keeps its own role-named Activity and package. Preserve the exact layout hierarchy in the appendix. The image-backed sticky header stays **outside** the `personal_content` ScrollView and has a fixed height of **172dp** so its title row and search trigger fit without allowing the image to affect measurement. Build it as a `FrameLayout` with `dashboard_header_background.xml`, `clipToOutline="true"`, and `outlineProvider="background"`; layer a `centerCrop` `personal_seats_3_alt` ImageView, `dashboard_header_image_overlay.xml`, and then the padded header content. The content keeps 22dp horizontal padding, page title/subtitle, 40dp solid bell, 40dp initials avatar, and a 50dp full-width rounded `menu_search_background.xml` search trigger 18dp below the title row. Use white for the title and bell and `?attr/rolePrimaryLight` for the subtitle. The overlay uses the Personal blue `roleOverlayStart` and `roleOverlayEnd`. Keep the bell to the left of the avatar. Do not add the QR control. All later sections belong inside `personal_content`, leaving this header fixed.

Use this exact shared overlay for both dashboard header images:

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <gradient
        android:angle="0"
        android:startColor="?attr/roleOverlayStart"
        android:endColor="?attr/roleOverlayEnd" />
</shape>
```

The required stacking order is `ImageView`, overlay `View`, content `LinearLayout`. Keep the image and overlay at `match_parent` within the fixed-height `FrameLayout`; use `wrap_content` only for the final content container. The admin header is 150dp and the personal header is 172dp. This structure is mandatory when transferring the sticky headers to another project.

The header search is a **tap target**, not an EditText that types in place. Tapping any part opens `bottom_sheet_personal_search.xml` as a bottom drawer about 90% of the screen height, expanded on show. Preserve its rounded `dashboard_sheet_background.xml`, top handle, title and X, outlined 48dp `admin_outline_background.xml` search field with the search icon on the right, wrapping text-only `AdminStatusChip` filters, two-column `sample_item_card_list.xml` RecyclerView, and `view_empty_list.xml` zero-result state. Show All, the four 3/5/6/8 Seaters category chips, then any other categories actually present in the item data; update results immediately for both typing and chip selection. A dashboard seat-tile tap opens this same drawer with its matching seat chip already checked, while the header search opens with All checked. Search item name, category, detail, and tag; keep newest-first order and update the empty state. Reuse `ItemAdapter` and the existing local item source. Do not add an Add button or an admin-only filter to this personal search drawer.

The current personal dashboard starts with the seat-category grid directly below the fixed header and has no large CTA banner. Add a banner only when a later request calls for one.

Directly below the fixed header, keep the **staggered seat-category grid** in `activity_personal_dashboard.xml`. It has two equal-width columns with a 10dp gutter and four visible `view_personal_seat_tile.xml` cards: 3 Seaters at top left (224dp high), 5 Seaters at top right (172dp), 6 Seaters at bottom left (172dp), and 8 Seaters at bottom right (224dp), with 10dp between each column's cards. This is the intentional unequal-height pattern from the reference; do not flatten it into equal grid cells or let it cover the sections below. Each rounded tile keeps its title as a fixed `TextView` above a `ViewFlipper` that fades between **two different vehicle photos**. Set `autoStart="false"` and `flipInterval="8000"`; start the four flippers in 3/5/6/8 order with 0/2/4/6-second offsets, then cancel pending starts and stop flipping in `onStop`. The two-second gap remains consistent across cycles. Do not bind the same photo twice or simulate the change with a zoomed crop. Set the fixed seat titles inline in XML. Copy all eight local binary assets from `drawable-nodpi`: `personal_seats_3.jpg` and `personal_seats_3_alt.jpg`, `personal_seats_5.jpg` and `personal_seats_5_alt.jpg`, `personal_seats_6.jpg` and `personal_seats_6_alt.jpg`, and `personal_seats_8.jpg` and `personal_seats_8_alt.jpg`. Make each whole seat tile tappable and open the item search drawer with the corresponding 3/5/6/8 Seaters chip selected. Filter the displayed items through the same category-chip path used by manual selection, and show the shared empty state when the current item source has no matching category.

Below the seat grid, keep **Featured Items first and Activities second**. Featured Items has the section heading, subheading, right-side count badge, then exactly **one horizontally scrolling RecyclerView row**. Use `FeaturedItemAdapter` in `adapters`, which reuses `ItemAdapter` binding and `sample_item_card_list.xml` with 240dp-wide cards; use a horizontal `LinearLayoutManager`. Never use a two-column grid, wrap to a second row, or add a featured-item search field. Keep cards in newest-first order. Use `view_empty_list.xml` if no featured items exist.

Activities has the matching heading/subheading/count badge below Featured Items, then its **own** outlined 48dp search field, search icon on the right, wrapping text-only `AdminStatusChip` group (All, Online, In person), and a vertical `sample_activity_card_list.xml` RecyclerView with `ActivityCardAdapter`. Search and chip selection filter the activity cards together; update the count and show `view_empty_list.xml` for zero results. Keep default newest-first order and do not let activity search affect Featured Items or the header search drawer. Bind counts from actual visible data, never from the XML preview placeholders. Copy the sample row XML, badge styles, `AdminStatusChipTheme`, font, typography, colors, and referenced drawables so the rendered cards match rather than substituting plain TextViews.

The personal bell and avatar use the **same interactions and resources as the admin header**. Bell tint uses `dashboard_bell_tint.xml`: neutral when empty, colored when notifications exist; show the pulsing destructive-red dot only when nonempty. Bell tap opens `bottom_sheet_notifications.xml` with `NotificationAdapter` and `sample_notification_list.xml`; X uses the danger confirmation modal and removal updates the sheet, empty state, and bell from the same list. Initials tap opens the anchored `popup_admin_account.xml` Profile/Logout menu. Profile opens the separate `PersonalProfileActivity` with `activity_personal_profile.xml`, using the shared profile design described below with personal data. Logout clears `StaticUserModel.user` and starts `SampleLoginActivity` directly. Register the profile Activity in the manifest. The account model is `StaticUserModel.user`; use it for session-aware identity data.

## Profile pages

Use separate `activity_admin_profile.xml` and `activity_personal_profile.xml` layouts and role-owned Activities. Each page has a back/title header above a scrollable white rounded card on `?attr/rolePageBackground`. The card starts with a 152dp role-gradient cover, an 80dp role-colored initials avatar overlapping its lower edge, left-aligned name/email, and one pale rounded Role/Access/Status facts panel using `PersonFactColumn`, `PersonFactLabel`, and `PersonFactValue`. Below a divider, show Public profile with labels above the name and email fields using `form_field_background.xml` and small inline error text. Another divider separates a Notifications preference and a Log out row. End with Cancel and a role-colored Save changes button. Keep the exact XML hierarchy, role attributes, Arial Rounded font, and fixed typography sizes.

The sample profile edits name, email, and the notification preference in local `SharedPreferences` for its role. Save only after the confirmation modal, validate name and email with inline errors, update the visible identity and initials after persistence, and show the approved toast after success. Cancel and back discard unsaved edits. Log out clears `StaticUserModel.user` and starts Login immediately, without a modal. Authentication uses the local users table.

## List pages and reusable rows

Name a list Activity `RoleEntityListActivity`, for example `AdminMenuListActivity` or `AdminPersonListActivity`. The page has a back/title/add header, rounded outlined search field with a search vector on the right, wrapping `ChipGroup` below it, RecyclerView, and the shared empty state. Use `activity_admin_menu_list.xml` and `activity_admin_person_list.xml` for exact XML. Chips are text-only: never put icons on chips. A filter group uses exactly one dimension. Choose it by priority: (1) Category, (2) meaningful workflow Status, (3) Type, (4) other attributes such as seats. **Active and Inactive never make Status eligible or preferred as a chip filter.** If those are the only status values, skip Status and choose Type or another useful attribute; if none exists, omit filter chips. Do not create an All/Active/Inactive chip group. Other meaningful statuses such as Pending, Approved, or Cancelled can make Status eligible. The current person-list snapshot still shows Active/Inactive; it is a legacy visual reference, and this rule overrides its filter choices for new or adapted screens. Use the highest-priority eligible dimension for that list; never mix category and status or other dimensions in one group. An `All` chip is allowed when a group exists. Render flexible values from the Activity when appropriate. Use `AdminStatusChipTheme`, `AdminStatusChip`, the color selectors in `res/color`, and `admin_outline_background.xml` rather than custom visual approximations. Add uses `AppCompatButton` with the existing gradient `item_action_background.xml`.

Match each record to its row type, exactly: people use `sample_person_list.xml` (white outlined card, avatar with no red profile dot, name/email, status/date, role/specialty tags, grouped fact panel, Edit/Delete quick actions); tangible items or animals use `sample_item_card_list.xml` (image-led item card); appointments, orders, requests, and events use `sample_activity_card_list.xml` (activity card). The personal featured section uses its specified 240dp `sample_item_card_list.xml` cards in one horizontal row. Reuse their background drawables, badge styles, and fonts. Do not substitute a plain TextView row. Default sorting is newest first. The shared empty layout appears when a filter or search finds nothing.

## Create/edit and show pages

Use `activity_sample_create_page.xml` for forms: back/title/Save header, white form card on `?attr/rolePageBackground`, folder heading, dashed upload area and contained image preview, two-column short fields, description area, date/time fields, spinner, text-only multi-select chips, status radio controls, checkbox, and small error text under required input. Reuse `form_field_background.xml`, `form_upload_background.xml`, the role-colored `form_action_solid.xml`, the style definitions consolidated into `styles.xml`, and the included vectors. Keep image scaling contained so the chosen photo is not cropped. Edit pages follow the same layout and prefill fields; do not redesign them.

Use `activity_admin_item_show.xml` for an item show page: edge-to-edge image hero, circular back/favorite controls over the photo, a white sheet overlapping the image with rounded top corners, left-aligned title, compact rating/location line, one pale rounded facts panel, and a fixed bottom price/share bar. Match `sample_person_list.xml` by using `person_facts_background.xml` and the `PersonFactColumn`, `PersonFactLabel`, and `PersonFactValue` styles: each of four columns has a muted label above a dark value, with no separate tile borders. Bind Category, Details, Tag, and Status from the item. Put an About section immediately below the facts panel, with a heading and the item description; use “About this vehicle” for vehicle items and “About this item” otherwise. Keep the Arial Rounded font and current role palette, and reuse the confirmation modal/toast for favorite changes. Use `activity_sample_activity_show.xml` for activity/order/appointment/rental details. Preserve its fixed back/reference header, scrollable icon and centered hero title/subtitle, separate white outlined cards, role-colored section headings, label/value rows with right-aligned values, role-colored pill badges, dividers, and comments/notes card. Rename the headings and bind domain values as needed, and add rows or cards for extra fields using the same `ActivityShowCard`, `ActivityShowHeading`, `ActivityShowRow`, `ActivityShowLabel`, `ActivityShowValue`, and `ActivityShowValueBadge` styles. Keep the hero visible at the initial scroll position. Never collapse a section such as payment/fees into one multiline TextView or replace structured rows with plain text. Bring over the referenced styles, badge drawables, icon assets, typography, font, and role attributes; use `?attr/rolePageBackground` for the page. A shared activity-show screen selects its theme from `StaticUserModel.user.role` before `super.onCreate`. Before finishing, compare the rendered page at the top and after scrolling with the sample XML/reference image and correct visible differences in hierarchy, color, spacing, alignment, and badges. Do not claim a visual match from compilation alone.

## Notifications, modal, toast, and reports

The header bell is solid, on the left of the avatar. It is plain/neutral when the notification list is empty; with notifications it uses `?attr/rolePrimary` and a pulsing destructive-red dot. Clicking opens `bottom_sheet_notifications.xml` with a RecyclerView. Each row uses `sample_notification_list.xml`: a `?attr/rolePrimaryDark` circle with the SAME white `ic_dashboard_bell` icon for every notification, title, message, relative time, and a right-side X. X opens the existing danger confirmation modal before removal. Feed the sheet from the same list used to set the bell state; update both after removal. This is the in-app notification drawer. The top toast is a separate in-app overlay defined by `toast_banner.xml`.

Reports use `activity_sample_report_page.xml`, `report_bar_row.xml`, and their existing drawables. Reports are admin-owned and must use `Theme.RestaurantApp.Admin`: black total card and PDF action, grayscale role accents, role-colored progress bars, and `?attr/rolePageBackground`. Never allow the default green palette to appear decoratively on the report. Keep compact data presentation. Every summary metric outside the large total card and chart uses a structured horizontal row: label on the left, value right-aligned at the opposite edge. Reuse a row XML/style for repeated metrics, including collections, fees, fleet, and staff counts; bind the individual TextViews from data. Do not assemble report sections as newline-separated text in one TextView or use spaces to align labels and values.

## Mandatory report download

Every report screen requested by the user must include a working **Download PDF** action, unless the user explicitly says the button is visual-only. A click must open Android's document picker with `ActivityResultContracts.CreateDocument("application/pdf")`, generate a real PDF using `android.graphics.pdf.PdfDocument`, and write the bytes to the returned URI through `getContentResolver().openOutputStream(uri)`. Do not leave the button unwired, show a preview-only toast, produce an empty file, or save a screenshot/image with a `.pdf` extension. The current `SampleReportPageActivity.java` is the minimal working reference; adapt its report content and package name to the destination project. A role-owned report Activity, such as `AdminReportActivity`, belongs in the `admin` package. A shared PDF helper, if needed for multiple screens, belongs in `helpers`, never in the base package.

The PDF must contain the report title, selected period/filter if present, totals, category/status breakdown, and actual displayed figures. Load figures from the app's current local data or its explicit sample seeder; do not export unrelated hardcoded numbers. Export the styled `reportContent` view so the PDF retains the screen's cards, palette, typography, chart, and left-label/right-value rows. Do not build a second plain-text `List<String>` report and draw its lines with `Canvas.drawText`; that produces a visually different document and can drift from the live screen. The simple sample draws the report content view onto an A4-sized `PdfDocument.Page` with a calculated scale. If content is too long to read at that scale, paginate it rather than silently clipping it. When choosing a page break from child-view edges, compare every edge against the original page-height limit and take the *last* edge that fits; shrinking the limit during the scan causes one section per page and large blank areas. Render and inspect every PDF page, checking that no card or text is clipped and pages are reasonably filled. On a canceled picker result, do nothing. On success, close the stream and PDF and show the approved success toast; on failure, close resources and show the approved error toast. Exporting a PDF itself does not require a danger confirmation modal because it does not change application data.

Minimum Activity pattern:

```java
import android.graphics.Canvas;
import android.graphics.pdf.PdfDocument;
import android.view.View;
import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import java.io.OutputStream;

ActivityResultLauncher<String> savePdf = registerForActivityResult(
        new ActivityResultContracts.CreateDocument("application/pdf"), uri -> {
            if (uri == null) return;
            View content = binding.reportContent;
            PdfDocument pdf = new PdfDocument();
            try (OutputStream out = getContentResolver().openOutputStream(uri)) {
                PdfDocument.Page page = pdf.startPage(
                        new PdfDocument.PageInfo.Builder(595, 842, 1).create());
                Canvas canvas = page.getCanvas();
                canvas.translate(24, 24);
                float scale = Math.min(547f / content.getWidth(), 794f / content.getHeight());
                canvas.scale(scale, scale);
                content.draw(canvas);
                pdf.finishPage(page);
                pdf.writeTo(out);
                AppToast.success(this, "PDF saved", "Your report is ready.");
            } catch (Exception e) {
                AppToast.error(this, "Save failed", "Please try again.");
            } finally {
                pdf.close();
            }
        });
binding.reportDownload.setOnClickListener(v -> savePdf.launch("report.pdf"));
```

The report XML in the appendix provides `report_content` and `report_download`. Reuse `report_download_background.xml` and `ic_report_download.xml` for the exact button appearance. Before handoff, use the button on a device/emulator when available, choose a destination, and confirm the saved file opens as a nonempty PDF with the expected data. An offline build alone does not verify PDF output.

## Binary assets required for the exact appearance

XML cannot contain the bitmap and font bytes. Copy these files byte-for-byte into the same resource folders when reproducing the design. The seat grid and item cards need the images; all text needs the font. Launcher assets complete the app icon.

- `app/src/main/res/drawable-nodpi/personal_seats_3.jpg`
- `app/src/main/res/drawable-nodpi/personal_seats_3_alt.jpg`
- `app/src/main/res/drawable-nodpi/personal_seats_5.jpg`
- `app/src/main/res/drawable-nodpi/personal_seats_5_alt.jpg`
- `app/src/main/res/drawable-nodpi/personal_seats_6.jpg`
- `app/src/main/res/drawable-nodpi/personal_seats_6_alt.jpg`
- `app/src/main/res/drawable-nodpi/personal_seats_8.jpg`
- `app/src/main/res/drawable-nodpi/personal_seats_8_alt.jpg`
- `app/src/main/res/drawable-nodpi/sample_item_photo.png`
- `app/src/main/res/font/arial_rounded_mt_bold.ttf`
- `app/src/main/res/mipmap-hdpi/ic_launcher.webp`
- `app/src/main/res/mipmap-hdpi/ic_launcher_round.webp`
- `app/src/main/res/mipmap-mdpi/ic_launcher.webp`
- `app/src/main/res/mipmap-mdpi/ic_launcher_round.webp`
- `app/src/main/res/mipmap-xhdpi/ic_launcher.webp`
- `app/src/main/res/mipmap-xhdpi/ic_launcher_round.webp`
- `app/src/main/res/mipmap-xxhdpi/ic_launcher.webp`
- `app/src/main/res/mipmap-xxhdpi/ic_launcher_round.webp`
- `app/src/main/res/mipmap-xxxhdpi/ic_launcher.webp`
- `app/src/main/res/mipmap-xxxhdpi/ic_launcher_round.webp`

## Complete current XML resources

All paths below are relative to the project root. Each fenced block is the full file content at the time this guide was generated. The live XML files are authoritative if they change later. Keep resource names together when copying to a new project: a layout may require values, selectors, vectors, and shape drawables from multiple sections.

### res/layout (26 files)

#### `app/src/main/res/layout/activity_admin_dashboard.xml`

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
    android:background="@color/white"
    tools:context=".admin.AdminDashboardActivity">
    <LinearLayout
        android:id="@+id/admin_header"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/dashboard_header_background"
        android:elevation="8dp"
        android:orientation="vertical"
        android:paddingHorizontal="22dp"
        android:paddingTop="22dp"
        android:paddingBottom="18dp">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center_vertical"
            android:orientation="horizontal">
            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:orientation="vertical">
                <TextView
                    style="@style/AdminHeading"
                    android:text="@string/admin_hello" />
                <TextView
                    style="@style/AdminHeading"
                    android:text="@string/admin_name" />
            </LinearLayout>

            <FrameLayout
                android:id="@+id/admin_notifications_button"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:layout_marginStart="8dp"
                android:foreground="?attr/selectableItemBackgroundBorderless">
                <ImageView
                    android:id="@+id/admin_bell_icon"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:padding="8dp"
                    android:src="@drawable/ic_dashboard_bell"
                    android:tint="@color/dashboard_bell_tint" />
                <View
                    android:id="@+id/admin_notification_dot"
                    android:layout_width="8dp"
                    android:layout_height="8dp"
                    android:layout_gravity="top|end"
                    android:layout_margin="2dp"
                    android:background="@drawable/user_avatar_background"
                    android:backgroundTint="@color/destructive" />
            </FrameLayout>
            <TextView
                android:id="@+id/admin_avatar_button"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:layout_marginStart="8dp"
                android:background="@drawable/admin_avatar_background"
                android:clickable="true"
                android:contentDescription="@string/admin_account_menu"
                android:focusable="true"
                android:fontFamily="@font/arial_rounded_mt_bold"
                android:foreground="?attr/selectableItemBackgroundBorderless"
                android:gravity="center"
                android:text="@string/admin_initials"
                android:textColor="@color/primary_dark" />
        </LinearLayout>
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="6dp"
            android:text="@string/admin_subtitle"
            android:textColor="@color/text_subheading"
            android:textSize="@dimen/type_body" />
    </LinearLayout>

    <ScrollView
        android:id="@+id/admin_scroll"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:paddingHorizontal="22dp"
            android:paddingBottom="20dp">
            <ViewFlipper
                android:id="@+id/admin_feature"
                android:layout_width="match_parent"
                android:layout_height="166dp"
                android:layout_marginTop="16dp"
                android:autoStart="false"
                android:flipInterval="3900"
                android:inAnimation="@android:anim/fade_in"
                android:outAnimation="@android:anim/fade_out">
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:background="@drawable/admin_feature_background"
                    android:orientation="vertical"
                    android:padding="20dp">
                    <TextView
                        android:id="@+id/admin_feature_update_label"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="●  Update"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_body" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="12dp"
                        android:text="Feb 12th 2024"
                        android:textColor="@color/primary_light"
                        android:textSize="@dimen/type_label" />
                    <TextView
                        android:id="@+id/admin_feature_update_message"
                        android:layout_width="match_parent"
                        android:layout_height="0dp"
                        android:layout_marginTop="6dp"
                        android:layout_weight="1"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="Sales revenue increased 40% in 1 week"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_page_heading" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="See Statistics  ›"
                        android:textColor="@color/primary_light"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:background="@drawable/admin_feature_background"
                    android:backgroundTint="@color/primary_dark"
                    android:orientation="vertical"
                    android:padding="20dp">
                    <TextView
                        android:id="@+id/admin_feature_items_label"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="●  Items"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_body" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="12dp"
                        android:text="Available now"
                        android:textColor="@color/primary_light"
                        android:textSize="@dimen/type_label" />
                    <TextView
                        android:id="@+id/admin_feature_items_message"
                        android:layout_width="match_parent"
                        android:layout_height="0dp"
                        android:layout_marginTop="6dp"
                        android:layout_weight="1"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        tools:text="3 items ready to browse"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_page_heading" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="See Statistics  ›"
                        android:textColor="@color/primary_light"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:background="@drawable/admin_feature_background"
                    android:backgroundTint="@color/badge_green"
                    android:orientation="vertical"
                    android:padding="20dp">
                    <TextView
                        android:id="@+id/admin_feature_activities_label"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="●  Activities"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_body" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="12dp"
                        android:text="Latest activity"
                        android:textColor="@color/primary_light"
                        android:textSize="@dimen/type_label" />
                    <TextView
                        android:id="@+id/admin_feature_activities_message"
                        android:layout_width="match_parent"
                        android:layout_height="0dp"
                        android:layout_marginTop="6dp"
                        android:layout_weight="1"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        tools:text="3 activities to review"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_page_heading" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="See Statistics  ›"
                        android:textColor="@color/primary_light"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
            </ViewFlipper>

            <LinearLayout
                android:id="@+id/admin_metrics"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:orientation="horizontal"
                android:padding="8dp"
                android:clipChildren="false"
                android:clipToPadding="false">
                <LinearLayout
                    android:id="@+id/admin_items_metric"
                    style="@style/AdminMetricFloatingCard"
                    android:clickable="true"
                    android:foreground="?attr/selectableItemBackground">
                    <TextView
                        style="@style/AdminMetricLabel"
                        android:text="@string/admin_items_metric_title" />
                    <TextView
                        android:id="@+id/admin_items_metric_value"
                        style="@style/AdminMetricValue"
                        android:text="3" />
                    <TextView
                        style="@style/AdminMetricChange"
                        android:text="@string/admin_items_metric_action"
                        android:textColor="@color/primary_dark" />
                </LinearLayout>
                <LinearLayout
                    android:id="@+id/admin_persons_metric"
                    style="@style/AdminMetricCard"
                    android:layout_marginStart="12dp"
                    android:clickable="true"
                    android:foreground="?attr/selectableItemBackground">
                    <TextView
                        style="@style/AdminMetricLabel"
                        android:text="@string/admin_persons_metric_title" />
                    <TextView
                        android:id="@+id/admin_persons_metric_value"
                        style="@style/AdminMetricValue"
                        android:text="3" />
                    <TextView
                        style="@style/AdminMetricChange"
                        android:text="@string/admin_persons_metric_action"
                        android:textColor="@color/primary_dark" />
                </LinearLayout>
            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="48dp"
                android:layout_marginTop="18dp"
                android:orientation="horizontal"
                android:gravity="center_vertical"
                android:background="@drawable/admin_outline_background">
                <EditText
                    android:id="@+id/admin_search"
                    android:layout_width="0dp"
                    android:layout_height="match_parent"
                    android:layout_weight="1"
                    android:paddingStart="18dp"
                    android:paddingEnd="4dp"
                    android:background="@android:color/transparent"
                    android:hint="@string/admin_search_hint"
                    android:singleLine="true"
                    android:inputType="text"
                    android:textColor="@color/search_text"
                    android:textColorHint="@color/search_hint"
                    android:textSize="@dimen/type_body" />
                <ImageView
                    android:layout_width="42dp"
                    android:layout_height="match_parent"
                    android:padding="12dp"
                    android:src="@drawable/ic_list_search"
                    android:tint="@color/text_heading" />
            </LinearLayout>
            <com.google.android.material.chip.ChipGroup
                android:id="@+id/admin_statuses"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                app:singleLine="false"
                app:singleSelection="true"
                app:selectionRequired="false"
                app:chipSpacingHorizontal="8dp"
                app:chipSpacingVertical="8dp" />

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="18dp"
                android:gravity="center_vertical"
                android:orientation="horizontal">
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/admin_sample_items"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_section_heading" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="3dp"
                        android:text="@string/admin_sample_items_subheading"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
                <TextView
                    android:id="@+id/admin_transaction_count1"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:background="@drawable/menu_search_background"
                    android:gravity="center"
                    android:minWidth="42dp"
                    android:paddingHorizontal="12dp"
                    android:paddingVertical="7dp"
                    android:text="@string/admin_zero_items"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/admin_transactions_list1"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:nestedScrollingEnabled="false"
                android:visibility="visible" />
            <include
                android:id="@+id/admin_items_empty"
                android:visibility="gone"
                layout="@layout/view_empty_list" />

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="18dp"
                android:gravity="center_vertical"
                android:orientation="horizontal">
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/admin_sample_activities"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_section_heading" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="3dp"
                        android:text="@string/admin_sample_activities_subheading"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
                <TextView
                    android:id="@+id/admin_transaction_count"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:background="@drawable/menu_search_background"
                    android:gravity="center"
                    android:minWidth="42dp"
                    android:paddingHorizontal="12dp"
                    android:paddingVertical="7dp"
                    android:text="@string/admin_zero_items"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="48dp"
                android:layout_marginTop="14dp"
                android:background="@drawable/admin_outline_background"
                android:gravity="center_vertical"
                android:orientation="horizontal">
                <EditText
                    android:id="@+id/admin_activities_search"
                    android:layout_width="0dp"
                    android:layout_height="match_parent"
                    android:layout_weight="1"
                    android:background="@android:color/transparent"
                    android:hint="@string/admin_activities_search_hint"
                    android:inputType="text"
                    android:paddingStart="18dp"
                    android:paddingEnd="4dp"
                    android:singleLine="true"
                    android:textColor="@color/search_text"
                    android:textColorHint="@color/search_hint"
                    android:textSize="@dimen/type_body" />
                <ImageView
                    android:layout_width="42dp"
                    android:layout_height="match_parent"
                    android:padding="12dp"
                    android:src="@drawable/ic_list_search"
                    android:tint="@color/text_heading" />
            </LinearLayout>
            <com.google.android.material.chip.ChipGroup
                android:id="@+id/admin_activity_filters"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                app:chipSpacingHorizontal="8dp"
                app:chipSpacingVertical="8dp"
                app:selectionRequired="true"
                app:singleLine="false"
                app:singleSelection="true" />
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/admin_transactions_list"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:nestedScrollingEnabled="false"
                android:visibility="visible" />
            <include
                android:id="@+id/admin_activities_empty"
                android:visibility="gone"
                layout="@layout/view_empty_list" />
        </LinearLayout>
    </ScrollView>

</LinearLayout>
```

#### `app/src/main/res/layout/activity_admin_item_show.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    android:orientation="vertical">
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true"
        android:overScrollMode="never">
        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <FrameLayout
                android:layout_width="match_parent"
                android:layout_height="320dp">
                <ImageView
                    android:id="@+id/item_show_image"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:contentDescription="@null"
                    android:scaleType="centerCrop"
                    android:src="@drawable/sample_item_photo" />
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_gravity="top"
                    android:gravity="center_vertical"
                    android:orientation="horizontal"
                    android:paddingHorizontal="20dp"
                    android:paddingTop="20dp">
                    <androidx.appcompat.widget.AppCompatImageButton
                        android:id="@+id/item_show_back"
                        android:layout_width="44dp"
                        android:layout_height="44dp"
                        android:background="@drawable/item_show_icon_background"
                        android:contentDescription="@string/item_show_close"
                        android:padding="12dp"
                        android:src="@drawable/ic_list_back"
                        app:tint="@color/text_heading" />
                    <View
                        android:layout_width="0dp"
                        android:layout_height="1dp"
                        android:layout_weight="1" />
                    <androidx.appcompat.widget.AppCompatImageButton
                        android:id="@+id/item_show_favorite"
                        android:layout_width="44dp"
                        android:layout_height="44dp"
                        android:background="@drawable/item_show_icon_background"
                        android:contentDescription="@string/item_show_favorite"
                        android:padding="12dp"
                        android:src="@drawable/ic_item_heart"
                        app:tint="@color/destructive" />
                </LinearLayout>
            </FrameLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="292dp"
                android:background="@drawable/item_show_sheet_background"
                android:orientation="vertical"
                android:paddingHorizontal="24dp"
                android:paddingTop="30dp"
                android:paddingBottom="28dp">
                <TextView
                    android:id="@+id/item_show_name"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:fontFamily="@font/arial_rounded_mt_bold"
                    android:text="@string/sample_item_name"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_large_heading" />
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:gravity="center_vertical"
                    android:orientation="horizontal">
                    <TextView
                        android:id="@+id/item_show_rating"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:drawableStart="@drawable/ic_item_star"
                        android:drawablePadding="5dp"
                        android:text="@string/sample_item_rating"
                        android:textColor="@color/text_body"
                        android:textSize="@dimen/type_subheading" />
                    <TextView
                        android:id="@+id/item_show_location"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_marginStart="16dp"
                        android:layout_weight="1"
                        android:drawableStart="@drawable/ic_user_location"
                        android:drawablePadding="5dp"
                        android:singleLine="true"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="14dp"
                    android:background="@drawable/person_facts_background"
                    android:orientation="horizontal"
                    android:paddingHorizontal="10dp"
                    android:paddingVertical="12dp">
                    <LinearLayout
                        style="@style/PersonFactColumn">
                        <TextView
                            style="@style/PersonFactLabel"
                            android:text="@string/item_show_category_label" />
                        <TextView
                            android:id="@+id/item_show_category"
                            style="@style/PersonFactValue"
                            android:text="@string/sample_item_category" />
                    </LinearLayout>
                    <LinearLayout
                        style="@style/PersonFactColumn">
                        <TextView
                            style="@style/PersonFactLabel"
                            android:text="@string/item_show_detail_label" />
                        <TextView
                            android:id="@+id/item_show_detail"
                            style="@style/PersonFactValue"
                            android:text="@string/sample_item_detail" />
                    </LinearLayout>
                    <LinearLayout
                        style="@style/PersonFactColumn">
                        <TextView
                            style="@style/PersonFactLabel"
                            android:text="@string/item_show_tag_label" />
                        <TextView
                            android:id="@+id/item_show_tag"
                            style="@style/PersonFactValue" />
                    </LinearLayout>
                    <LinearLayout
                        style="@style/PersonFactColumn">
                        <TextView
                            style="@style/PersonFactLabel"
                            android:text="@string/item_show_status_label" />
                        <TextView
                            android:id="@+id/item_show_status"
                            style="@style/PersonFactValue" />
                    </LinearLayout>
                </LinearLayout>
                <TextView
                    android:id="@+id/item_show_about_heading"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="30dp"
                    android:fontFamily="@font/arial_rounded_mt_bold"
                    android:text="@string/item_show_about"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_card_heading" />
                <TextView
                    android:id="@+id/item_show_description"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:lineSpacingExtra="3dp"
                    android:text="@string/item_show_sample_description"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_body" />
            </LinearLayout>
        </FrameLayout>
    </ScrollView>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/white"
        android:elevation="12dp"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        android:paddingHorizontal="20dp"
        android:paddingVertical="14dp">
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical">
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/item_show_from"
                android:textColor="@color/text_subheading"
                android:textSize="@dimen/type_caption" />
            <TextView
                android:id="@+id/item_show_price"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="3dp"
                android:fontFamily="@font/arial_rounded_mt_bold"
                android:text="@string/sample_item_price"
                android:textColor="@color/primary_dark"
                android:textSize="@dimen/type_card_heading" />
        </LinearLayout>
        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/item_show_share"
            android:layout_width="154dp"
            android:layout_height="52dp"
            android:background="@drawable/item_action_background"
            android:text="@string/item_show_share"
            android:textAllCaps="false"
            android:textColor="@color/white"
            android:textSize="@dimen/type_button" />
    </LinearLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/activity_admin_menu_list.xml`

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
    android:background="@color/white"
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
            android:textSize="@dimen/type_page_heading"
            android:textStyle="bold"
            android:textColor="@color/text_heading" />
        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/menu_add"
            android:layout_width="wrap_content"
            android:layout_height="44dp"
            android:background="@drawable/item_action_background"
            android:drawableStart="@drawable/ic_list_plus"
            android:drawablePadding="6dp"
            android:drawableTint="@color/white"
            android:minWidth="0dp"
            android:paddingHorizontal="12dp"
            android:text="@string/menu_add"
            android:textSize="@dimen/type_label"
            android:textColor="@color/white"
            android:textAllCaps="false" />
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:layout_marginTop="18dp"
        android:orientation="horizontal"
        android:gravity="center_vertical"
        android:background="@drawable/admin_outline_background">
        <EditText
            android:id="@+id/menu_search"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:paddingStart="18dp"
            android:paddingEnd="4dp"
            android:background="@android:color/transparent"
            android:hint="@string/menu_search"
            android:singleLine="true"
            android:inputType="text"
            android:textColor="@color/search_text"
            android:textColorHint="@color/search_hint"
            android:textSize="@dimen/type_body" />
        <ImageView
            android:layout_width="42dp"
            android:layout_height="match_parent"
            android:padding="12dp"
            android:src="@drawable/ic_list_search"
            android:tint="@color/text_heading" />
    </LinearLayout>
    <com.google.android.material.chip.ChipGroup
        android:id="@+id/menu_categories"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:layout_marginBottom="8dp"
        app:singleLine="false"
        app:singleSelection="true"
        app:selectionRequired="true"
        app:chipSpacingHorizontal="8dp"
        app:chipSpacingVertical="8dp">
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_all"
            style="@style/AdminStatusChip"
            android:text="@string/menu_all"
            android:checked="true" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_food"
            style="@style/AdminStatusChip"
            android:text="@string/menu_food" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_drinks"
            style="@style/AdminStatusChip"
            android:text="@string/menu_drinks" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_desserts"
            style="@style/AdminStatusChip"
            android:text="@string/menu_desserts" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_breakfast"
            style="@style/AdminStatusChip"
            android:text="@string/menu_breakfast" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_lunch"
            style="@style/AdminStatusChip"
            android:text="@string/menu_lunch" />
        <com.google.android.material.chip.Chip
            android:id="@+id/filter_dinner"
            style="@style/AdminStatusChip"
            android:text="@string/menu_dinner" />
    </com.google.android.material.chip.ChipGroup>
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/menu_list"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:clipToPadding="false"
            android:paddingBottom="16dp"
            tools:listitem="@layout/sample_item_card_list"
            tools:itemCount="3"
            app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
            app:spanCount="2" />
        <include
            android:id="@+id/menu_empty"
            layout="@layout/view_empty_list"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="gone" />
    </FrameLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/activity_admin_person_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    android:orientation="vertical"
    android:paddingHorizontal="16dp"
    tools:context=".admin.AdminPersonListActivity">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="64dp"
        android:gravity="center_vertical"
        android:orientation="horizontal">
        <ImageView
            android:id="@+id/person_back"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:padding="8dp"
            android:src="@drawable/ic_list_back" />
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="@string/person_list_title"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_page_heading"
            android:textStyle="bold" />
        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/person_add"
            android:layout_width="wrap_content"
            android:layout_height="44dp"
            android:background="@drawable/item_action_background"
            android:drawableStart="@drawable/ic_list_plus"
            android:drawablePadding="6dp"
            android:drawableTint="@color/white"
            android:minWidth="0dp"
            android:paddingHorizontal="12dp"
            android:text="@string/person_add"
            android:textAllCaps="false"
            android:textColor="@color/white"
            android:textSize="@dimen/type_label" />
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:layout_marginTop="18dp"
        android:orientation="horizontal"
        android:gravity="center_vertical"
        android:background="@drawable/admin_outline_background">
        <EditText
            android:id="@+id/person_search"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:paddingStart="18dp"
            android:paddingEnd="4dp"
            android:background="@android:color/transparent"
            android:hint="@string/person_search"
            android:singleLine="true"
            android:inputType="text"
            android:textColor="@color/search_text"
            android:textColorHint="@color/search_hint"
            android:textSize="@dimen/type_body" />
        <ImageView
            android:layout_width="42dp"
            android:layout_height="match_parent"
            android:padding="12dp"
            android:src="@drawable/ic_list_search"
            android:tint="@color/text_heading" />
    </LinearLayout>
    <com.google.android.material.chip.ChipGroup
        android:id="@+id/person_filters"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:layout_marginBottom="8dp"
        app:chipSpacingHorizontal="8dp"
        app:chipSpacingVertical="8dp"
        app:selectionRequired="true"
        app:singleLine="false"
        app:singleSelection="true" />
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/person_list"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:clipToPadding="false"
            android:paddingBottom="16dp"
            app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
            tools:itemCount="3"
            tools:listitem="@layout/sample_person_list" />
        <include
            android:id="@+id/person_empty"
            layout="@layout/view_empty_list"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="gone" />
    </FrameLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/activity_dashboard_sample.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@color/white"
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
            android:tint="@color/white" />
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
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_section_heading"
                android:textStyle="bold" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="@string/dashboard_panel"
                android:textColor="@color/text_subheading"
                android:textSize="@dimen/type_subheading" />
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
                android:backgroundTint="@color/destructive" />
        </FrameLayout>
        <TextView
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:layout_marginStart="10dp"
            android:background="@drawable/user_avatar_background"
            android:gravity="center"
            android:text="@string/dashboard_initials"
            android:textColor="@color/primary_dark"
            android:textStyle="bold"
            android:textSize="@dimen/type_body_large" />
    </LinearLayout>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:text="@string/dashboard_greeting"
        android:textSize="@dimen/type_metric"
        android:textStyle="bold"
        android:textColor="@color/text_heading" />
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="5dp"
        android:text="@string/dashboard_description"
        android:textSize="@dimen/type_subheading"
        android:textColor="@color/text_subheading" />
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
                        android:tint="@color/primary_dark" />
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
                        android:backgroundTint="@color/primary_light"
                        android:text="@string/dashboard_users_growth"
                        android:textColor="@color/primary_dark"
                        android:textStyle="bold"
                        android:textSize="@dimen/type_label" />
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
                        android:tint="@color/primary_dark" />
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
                        android:backgroundTint="@color/primary_light"
                        android:text="@string/dashboard_items_growth"
                        android:textColor="@color/primary_dark"
                        android:textStyle="bold"
                        android:textSize="@dimen/type_label" />
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
                        android:tint="@color/primary_dark" />
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
                        android:backgroundTint="@color/primary_light"
                        android:text="@string/dashboard_requests_growth"
                        android:textColor="@color/primary_dark"
                        android:textStyle="bold"
                        android:textSize="@dimen/type_label" />
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
                        android:tint="@color/primary_dark" />
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
                        android:backgroundTint="@color/primary_light"
                        android:text="@string/dashboard_completed_growth"
                        android:textColor="@color/primary_dark"
                        android:textStyle="bold"
                        android:textSize="@dimen/type_label" />
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
            android:tint="@color/primary_dark" />
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
                android:textSize="@dimen/type_card_heading"
                android:textStyle="bold"
                android:textColor="@color/text_heading" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="3dp"
                android:text="@string/dashboard_important_subtitle"
                android:textSize="@dimen/type_label"
                android:textColor="@color/text_subheading" />
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
            android:textSize="@dimen/type_subheading"
            android:textStyle="bold"
            android:textColor="@color/text_subheading" />
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

#### `app/src/main/res/layout/activity_sample_activity_show.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/activity_show_background"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="64dp"
        android:background="@color/white"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        android:paddingHorizontal="16dp">
        <ImageView
            android:id="@+id/activity_show_back"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:padding="8dp"
            android:src="@drawable/ic_list_back" />
        <TextView
            android:id="@+id/activity_show_reference"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="@string/activity_show_sample_reference"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_section_heading" />
        <View
            android:layout_width="40dp"
            android:layout_height="40dp" />
    </LinearLayout>
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:paddingHorizontal="16dp"
            android:paddingBottom="24dp">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:gravity="center_horizontal"
                android:orientation="vertical"
                android:paddingTop="24dp"
                android:paddingBottom="14dp">
                <ImageView
                    android:id="@+id/activity_show_icon"
                    android:layout_width="76dp"
                    android:layout_height="76dp"
                    android:background="@drawable/activity_show_icon_background"
                    android:contentDescription="@string/activity_show_icon_description"
                    android:elevation="5dp"
                    android:padding="20dp"
                    android:src="@drawable/ic_activity_summary" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="14dp"
                    android:gravity="center"
                    android:text="@string/activity_show_hero_title"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_page_heading" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:gravity="center"
                    android:text="@string/activity_show_hero_subtitle"
                    android:textColor="@color/text_subheading"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
            <LinearLayout style="@style/ActivityShowCard">
                <TextView
                    style="@style/ActivityShowHeading"
                    android:text="@string/activity_show_overview" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="18dp"
                    android:text="@string/activity_show_title_label"
                    android:textColor="@color/text_subheading"
                    android:textSize="@dimen/type_label" />
                <TextView
                    android:id="@+id/activity_show_title"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="3dp"
                    android:text="@string/sample_activity_title"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_body_large" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="16dp"
                    android:text="@string/activity_show_organizer_label"
                    android:textColor="@color/text_subheading"
                    android:textSize="@dimen/type_label" />
                <TextView
                    android:id="@+id/activity_show_organizer"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="3dp"
                    android:text="@string/sample_activity_subtitle"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_body" />
            </LinearLayout>
            <LinearLayout style="@style/ActivityShowCard">
                <TextView
                    style="@style/ActivityShowHeading"
                    android:text="@string/activity_show_schedule" />
                <LinearLayout style="@style/ActivityShowRow">
                    <TextView
                        style="@style/ActivityShowLabel"
                        android:text="@string/activity_show_date_label" />
                    <TextView
                        android:id="@+id/activity_show_date"
                        style="@style/ActivityShowValueBadge"
                        android:text="@string/sample_activity_date" />
                </LinearLayout>
                <LinearLayout style="@style/ActivityShowRow">
                    <TextView
                        style="@style/ActivityShowLabel"
                        android:text="@string/activity_show_time_label" />
                    <TextView
                        android:id="@+id/activity_show_time"
                        style="@style/ActivityShowValue"
                        android:text="@string/sample_activity_time" />
                </LinearLayout>
                <LinearLayout style="@style/ActivityShowRow">
                    <TextView
                        style="@style/ActivityShowLabel"
                        android:text="@string/activity_show_mode_label" />
                    <TextView
                        android:id="@+id/activity_show_mode"
                        style="@style/ActivityShowValueBadge"
                        android:text="@string/sample_activity_mode" />
                </LinearLayout>
            </LinearLayout>
            <LinearLayout style="@style/ActivityShowCard">
                <TextView
                    style="@style/ActivityShowHeading"
                    android:text="@string/activity_show_payment" />
                <LinearLayout style="@style/ActivityShowRow">
                    <TextView
                        style="@style/ActivityShowLabel"
                        android:text="@string/activity_show_payment_method" />
                    <TextView
                        style="@style/ActivityShowValue"
                        android:text="@string/activity_show_not_required" />
                </LinearLayout>
                <LinearLayout style="@style/ActivityShowRow">
                    <TextView
                        style="@style/ActivityShowLabel"
                        android:text="@string/activity_show_amount" />
                    <TextView
                        style="@style/ActivityShowValue"
                        android:text="@string/activity_show_dash" />
                </LinearLayout>
                <View
                    android:layout_width="match_parent"
                    android:layout_height="1dp"
                    android:layout_marginTop="14dp"
                    android:background="@color/search_border" />
                <LinearLayout style="@style/ActivityShowRow">
                    <TextView
                        style="@style/ActivityShowLabel"
                        android:text="@string/activity_show_total" />
                    <TextView
                        style="@style/ActivityShowValue"
                        android:text="@string/activity_show_dash" />
                </LinearLayout>
            </LinearLayout>
            <LinearLayout style="@style/ActivityShowCard">
                <TextView
                    style="@style/ActivityShowHeading"
                    android:text="@string/activity_show_notes" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="14dp"
                    android:lineSpacingExtra="4dp"
                    android:text="@string/activity_show_no_notes"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
        </LinearLayout>
    </ScrollView>
</LinearLayout>
```

#### `app/src/main/res/layout/activity_sample_create_page.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/create_page_background"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="64dp"
        android:background="@color/white"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        android:paddingHorizontal="16dp">
        <ImageView
            android:id="@+id/create_back"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:padding="8dp"
            android:src="@drawable/ic_list_back" />
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="@string/create_page_title"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_section_heading"
            android:textStyle="bold" />
        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/create_save"
            android:layout_width="wrap_content"
            android:layout_height="40dp"
            android:background="@drawable/form_action_solid"
            android:paddingHorizontal="14dp"
            android:text="@string/create_save"
            android:textAllCaps="false"
            android:textColor="@color/white"
            android:textSize="@dimen/type_label" />
    </LinearLayout>
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="12dp"
            android:background="@drawable/admin_outline_background"
            android:elevation="2dp"
            android:orientation="vertical"
            android:padding="16dp">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:gravity="center_vertical"
                android:orientation="horizontal">
                <ImageView
                    android:layout_width="22dp"
                    android:layout_height="22dp"
                    android:src="@drawable/ic_form_folder" />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="8dp"
                    android:text="@string/create_page_heading"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_card_heading"
                    android:textStyle="bold" />
            </LinearLayout>
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="4dp"
                android:text="@string/create_page_subtitle"
                android:textColor="@color/text_subheading"
                android:textSize="@dimen/type_label" />
            <View
                android:layout_width="match_parent"
                android:layout_height="1dp"
                android:layout_marginTop="16dp"
                android:layout_marginBottom="14dp"
                android:background="@color/page_divider" />
            <FrameLayout
                android:id="@+id/create_image_picker"
                android:layout_width="match_parent"
                android:layout_height="104dp"
                android:background="@drawable/form_upload_background"
                android:clipToOutline="true">
                <ImageView
                    android:id="@+id/create_image"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:scaleType="fitCenter"
                    android:visibility="gone" />
                <LinearLayout
                    android:id="@+id/create_image_prompt"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:gravity="center"
                    android:orientation="vertical">
                    <ImageView
                        android:layout_width="28dp"
                        android:layout_height="28dp"
                        android:src="@drawable/ic_form_upload" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="5dp"
                        android:gravity="center"
                        android:text="@string/create_image_hint"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_subheading" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="3dp"
                        android:gravity="center"
                        android:text="@string/create_image_note"
                        android:textColor="@color/field_hint"
                        android:textSize="@dimen/type_detail" />
                </LinearLayout>
            </FrameLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        style="@style/CreateLabel"
                        android:text="@string/create_name_label" />
                    <EditText
                        android:id="@+id/create_name"
                        style="@style/CreateField"
                        android:hint="@string/create_name_hint"
                        android:inputType="textCapSentences"
                        android:singleLine="true" />
                    <TextView
                        android:id="@+id/create_name_error"
                        style="@style/CreateFieldError"
                        android:text="@string/create_name_required"
                        android:visibility="gone"
                        tools:visibility="visible" />
                </LinearLayout>
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="10dp"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        style="@style/CreateLabel"
                        android:text="@string/create_date_label" />
                    <EditText
                        android:id="@+id/create_date"
                        style="@style/CreateField"
                        android:drawableStart="@drawable/ic_user_calendar"
                        android:drawablePadding="4dp"
                        android:drawableTint="@color/field_icon"
                        android:focusable="false"
                        android:hint="@string/create_date_hint" />
                </LinearLayout>
            </LinearLayout>
            <TextView
                style="@style/CreateLabel"
                android:text="@string/create_description_label" />
            <EditText
                android:id="@+id/create_description"
                style="@style/CreateField"
                android:layout_height="88dp"
                android:gravity="top"
                android:hint="@string/create_description_hint"
                android:inputType="textCapSentences|textMultiLine"
                android:paddingVertical="10dp" />
            <TextView
                style="@style/CreateLabel"
                android:text="@string/create_tags_label" />
            <com.google.android.material.chip.ChipGroup
                android:id="@+id/create_tags"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="6dp"
                app:chipSpacingHorizontal="8dp"
                app:chipSpacingVertical="8dp"
                app:singleLine="false"
                app:singleSelection="false">
                <com.google.android.material.chip.Chip
                    android:id="@+id/create_tag_design"
                    style="@style/AdminStatusChip"
                    android:checked="true"
                    android:text="@string/create_tag_design" />
                <com.google.android.material.chip.Chip
                    android:id="@+id/create_tag_marketing"
                    style="@style/AdminStatusChip"
                    android:checked="true"
                    android:text="@string/create_tag_marketing" />
                <com.google.android.material.chip.Chip
                    android:id="@+id/create_tag_documentation"
                    style="@style/AdminStatusChip"
                    android:text="@string/create_tag_documentation" />
            </com.google.android.material.chip.ChipGroup>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        style="@style/CreateLabel"
                        android:text="@string/create_number_label" />
                    <EditText
                        android:id="@+id/create_number"
                        style="@style/CreateField"
                        android:hint="@string/create_number_hint"
                        android:inputType="numberDecimal"
                        android:singleLine="true" />
                </LinearLayout>
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="10dp"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        style="@style/CreateLabel"
                        android:text="@string/create_time_label" />
                    <EditText
                        android:id="@+id/create_time"
                        style="@style/CreateField"
                        android:drawableStart="@drawable/ic_activity_clock"
                        android:drawablePadding="4dp"
                        android:drawableTint="@color/field_icon"
                        android:focusable="false"
                        android:hint="@string/create_time_hint" />
                </LinearLayout>
            </LinearLayout>
            <TextView
                style="@style/CreateLabel"
                android:text="@string/create_option_label" />
            <Spinner
                android:id="@+id/create_category"
                style="@style/CreateField"
                android:entries="@array/create_categories" />
            <TextView
                style="@style/CreateLabel"
                android:text="@string/create_radio_label" />
            <RadioGroup
                android:id="@+id/create_status"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="2dp"
                android:checkedButton="@id/create_status_active"
                android:orientation="horizontal">
                <RadioButton
                    android:id="@+id/create_status_active"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:buttonTint="@color/success"
                    android:text="@string/create_radio_active"
                    android:textSize="@dimen/type_subheading" />
                <RadioButton
                    android:id="@+id/create_status_inactive"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="18dp"
                    android:buttonTint="@color/success"
                    android:text="@string/create_radio_inactive"
                    android:textSize="@dimen/type_subheading" />
            </RadioGroup>
            <CheckBox
                android:id="@+id/create_visible"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="4dp"
                android:buttonTint="@color/success"
                android:checked="true"
                android:text="@string/create_checkbox"
                android:textSize="@dimen/type_subheading" />
        </LinearLayout>
    </ScrollView>
</LinearLayout>
```

#### `app/src/main/res/layout/activity_sample_report_page.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    android:fitsSystemWindows="true"
    android:orientation="vertical">
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="64dp">
        <ImageButton
            android:id="@+id/report_back"
            android:layout_width="44dp"
            android:layout_height="44dp"
            android:layout_gravity="start|center_vertical"
            android:layout_marginStart="20dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:src="@drawable/ic_list_back" />
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="center"
            android:fontFamily="@font/arial_rounded_mt_bold"
            android:gravity="center"
            android:text="@string/report_title"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_page_heading" />
    </FrameLayout>
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">
        <LinearLayout
            android:id="@+id/report_content"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:paddingHorizontal="24dp"
            android:paddingTop="18dp"
            android:paddingBottom="24dp">
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:fontFamily="@font/arial_rounded_mt_bold"
                android:text="@string/report_overview"
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_report_title" />
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="4dp"
                android:text="@string/report_overview_subtitle"
                android:textColor="@color/text_subheading"
                android:textSize="@dimen/type_body" />
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="24dp"
                android:background="@drawable/admin_feature_background"
                android:orientation="vertical"
                android:padding="24dp">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="@string/report_total_count"
                    android:textColor="@color/primary_light"
                    android:textSize="@dimen/type_button" />
                <TextView
                    android:id="@+id/report_total"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:fontFamily="@font/arial_rounded_mt_bold"
                    android:text="@string/report_sample_total"
                    android:textColor="@color/white"
                    android:textSize="@dimen/type_report_total" />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="2dp"
                    android:text="@string/report_total_caption"
                    android:textColor="@color/primary_light"
                    android:textSize="@dimen/type_subheading" />
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="18dp"
                    android:text="@string/report_sample_label"
                    android:textColor="@color/primary_lime"
                    android:textSize="@dimen/type_label" />
            </LinearLayout>
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="28dp"
                android:fontFamily="@font/arial_rounded_mt_bold"
                android:text="@string/report_breakdown"
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_report_section" />
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginTop="4dp"
                android:text="@string/report_breakdown_subtitle"
                android:textColor="@color/text_subheading"
                android:textSize="@dimen/type_subheading" />
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
                <include
                    android:id="@+id/report_completed"
                    layout="@layout/report_bar_row" />
                <include
                    android:id="@+id/report_progress"
                    layout="@layout/report_bar_row" />
                <include
                    android:id="@+id/report_cancelled"
                    layout="@layout/report_bar_row" />
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="30dp"
                android:background="@drawable/admin_outline_background"
                android:orientation="vertical"
                android:padding="20dp">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:fontFamily="@font/arial_rounded_mt_bold"
                    android:text="@string/report_note_title"
                    android:textColor="@color/primary_dark"
                    android:textSize="@dimen/type_body_large" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="6dp"
                    android:text="@string/report_note"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
        </LinearLayout>
    </ScrollView>
    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/report_download"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:layout_marginHorizontal="24dp"
        android:layout_marginTop="8dp"
        android:layout_marginBottom="16dp"
        android:background="@drawable/report_download_background"
        android:elevation="0dp"
        android:fontFamily="@font/arial_rounded_mt_bold"
        android:gravity="center"
        android:paddingHorizontal="20dp"
        android:stateListAnimator="@null"
        android:text="@string/report_download_pdf"
        android:textAllCaps="false"
        android:textColor="@color/white"
        android:textSize="@dimen/type_body_large" />
</LinearLayout>
```

#### `app/src/main/res/layout/admin_transaction_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="76dp"
    android:orientation="horizontal"
    android:gravity="center_vertical"
    android:background="@drawable/admin_transaction_row_background">
    <ImageView
        android:id="@+id/transaction_icon"
        android:layout_width="48dp"
        android:layout_height="48dp"
        android:background="@drawable/admin_avatar_background"
        android:padding="12dp"
        android:tint="@color/primary_darkest"
        tools:src="@drawable/ic_admin_shirt" />
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:layout_marginStart="10dp"
        android:orientation="vertical">
        <TextView
            android:id="@+id/transaction_title"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:ellipsize="end"
            android:maxLines="1"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_body"
            android:fontFamily="@font/arial_rounded_mt_bold"
            tools:text="Tinek Detstar T-Shirt" />
        <TextView
            android:id="@+id/transaction_date"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:textColor="@color/text_subheading"
            android:textSize="@dimen/type_label"
            tools:text="Jul 12th 2024" />
    </LinearLayout>
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:gravity="end">
        <TextView
            android:id="@+id/transaction_status"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/text_body"
            android:textSize="@dimen/type_label"
            tools:text="Completed" />
        <TextView
            android:id="@+id/transaction_reference"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:textColor="@color/text_subheading"
            android:textSize="@dimen/type_detail"
            tools:text="OJWEJS7ISNC" />
    </LinearLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/bottom_sheet_notifications.xml`

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
        android:backgroundTint="@color/primary_light" />
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
            android:textSize="@dimen/type_page_heading"
            android:textStyle="bold"
            android:textColor="@color/text_heading" />
        <TextView
            android:id="@+id/notifications_close"
            android:layout_width="36dp"
            android:layout_height="36dp"
            android:background="@drawable/user_detail_background"
            android:gravity="center"
            android:text="×"
            android:textSize="@dimen/type_large_heading"
            android:textColor="@color/text_body" />
    </LinearLayout>
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="440dp"
        android:layout_marginTop="16dp">
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/notifications_list"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:clipToPadding="false"
            android:paddingBottom="8dp" />
        <include
            android:id="@+id/notifications_empty"
            layout="@layout/view_empty_list"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="gone" />
    </FrameLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/modal_status.xml`

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
        android:textColor="@color/white"
        android:textSize="@dimen/type_modal_symbol"
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
        android:textSize="@dimen/type_page_heading"
        android:textStyle="bold"
        android:textColor="@color/text_heading"
        android:accessibilityHeading="true"
        tools:text="Confirm your action" />
    <TextView
        android:id="@+id/modal_message"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:gravity="center"
        android:textSize="@dimen/type_button"
        android:lineSpacingExtra="3dp"
        android:textColor="@color/text_body"
        tools:text="Are you sure you want to continue? Confirm to complete this action." />
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="28dp"
        android:orientation="horizontal">
        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/modal_cancel"
            android:layout_width="0dp"
            android:layout_height="52dp"
            android:layout_weight="1"
            android:layout_marginEnd="12dp"
            android:background="@drawable/modal_cancel"
            android:text="@android:string/cancel"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_button"
            android:textAllCaps="false" />
        <androidx.appcompat.widget.AppCompatButton
            android:id="@+id/modal_confirm"
            android:layout_width="0dp"
            android:layout_height="52dp"
            android:layout_weight="1"
            android:background="@drawable/modal_confirm"
            android:backgroundTint="@null"
            android:elevation="6dp"
            android:textColor="@color/white"
            android:textSize="@dimen/type_button"
            android:textAllCaps="false"
            tools:text="@string/confirm_arrow" />
    </LinearLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/report_bar_row.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="18dp"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <TextView
            android:id="@+id/report_bar_label"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textColor="@color/text_body"
            android:textSize="@dimen/type_button" />
        <TextView
            android:id="@+id/report_bar_value"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:fontFamily="@font/arial_rounded_mt_bold"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_button" />
    </LinearLayout>
    <ProgressBar
        android:id="@+id/report_bar"
        style="@android:style/Widget.ProgressBar.Horizontal"
        android:layout_width="match_parent"
        android:layout_height="12dp"
        android:layout_marginTop="8dp"
        android:progressBackgroundTint="@color/primary_light" />
</LinearLayout>
```

#### `app/src/main/res/layout/sample_activity_card_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginVertical="8dp"
    app:cardCornerRadius="20dp"
    app:cardElevation="0dp"
    app:strokeWidth="1dp"
    app:strokeColor="@color/primary_light"
    app:cardBackgroundColor="@color/white">
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
                android:tint="@color/primary_dark" />
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
                    android:textSize="@dimen/type_body_large"
                    android:textStyle="bold"
                    android:textColor="@color/text_heading" />
                <TextView
                    android:id="@+id/activity_subtitle"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:text="@string/sample_activity_subtitle"
                    android:textSize="@dimen/type_label"
                    android:textColor="@color/text_subheading" />
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
                style="@style/green_badge"
                android:text="@string/sample_activity_type" />
            <TextView
                android:id="@+id/activity_date"
                style="@style/UserDetail"
                android:text="@string/sample_activity_date" />
            <TextView
                android:id="@+id/activity_time"
                style="@style/UserDetail"
                android:text="@string/sample_activity_time" />
            <TextView
                android:id="@+id/activity_mode"
                style="@style/UserDetail"
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
                android:backgroundTint="@color/primary_light"
                android:gravity="center"
                android:text="@string/sample_activity_initials"
                android:textStyle="bold"
                android:textSize="@dimen/type_label"
                android:textColor="@color/primary_dark" />
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
                    android:textSize="@dimen/type_label"
                    android:textStyle="bold"
                    android:textColor="@color/primary_dark" />
                <TextView
                    android:id="@+id/activity_organizer_role"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="2dp"
                    android:text="@string/sample_activity_role"
                    android:textSize="@dimen/type_caption"
                    android:textColor="@color/text_subheading" />
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
                android:textColor="@color/text_body"
                android:textSize="@dimen/type_detail"
                android:textAllCaps="false"
                app:cornerRadius="100dp"
                app:strokeColor="@color/primary_light"
                app:strokeWidth="1dp"
                app:backgroundTint="@color/primary_light" />
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
                android:textColor="@color/white"
                android:textSize="@dimen/type_detail"
                android:textStyle="bold"
                android:textAllCaps="false"
                app:icon="@drawable/ic_user_chevron"
                app:iconTint="@color/white"
                app:iconSize="14dp"
                app:iconGravity="textEnd"
                app:iconPadding="6dp" />
        </LinearLayout>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

#### `app/src/main/res/layout/sample_item_card_list.xml`

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
    app:strokeColor="@color/primary_light"
    app:cardBackgroundColor="@color/white"
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
                android:backgroundTint="@color/primary_light"
                android:src="@drawable/ic_item_heart" />
            <TextView
                android:id="@+id/item_status"
                style="@style/green_badge"
                android:layout_gravity="top|start"
                android:text="@string/item_available"
                android:textSize="@dimen/type_micro"
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
                android:textSize="@dimen/type_body"
                android:textStyle="bold"
                android:textColor="@color/text_heading"
                android:maxLines="2"
                android:ellipsize="end" />
            <TextView
                android:id="@+id/item_category"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="2dp"
                android:text="@string/sample_item_category"
                android:textSize="@dimen/type_detail"
                android:textColor="@color/text_subheading" />
            <com.google.android.material.chip.ChipGroup
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="6dp"
                app:chipSpacingHorizontal="6dp"
                app:chipSpacingVertical="4dp">
                <TextView
                    android:id="@+id/item_detail"
                    style="@style/ItemChip.Info"
                    android:text="@string/sample_item_detail" />
                <TextView
                    android:id="@+id/item_tag"
                    style="@style/ItemChip.Tag"
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
                    android:textSize="@dimen/type_caption"
                    android:textColor="@color/text_body" />
                <View
                    android:layout_width="1dp"
                    android:layout_height="10dp"
                    android:layout_marginHorizontal="5dp"
                    android:background="@color/primary_light" />
                <TextView
                    android:id="@+id/item_location"
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:text="@string/sample_item_location"
                    android:textSize="@dimen/type_caption"
                    android:textColor="@color/text_body" />
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
                    android:textSize="@dimen/type_body_large"
                    android:textStyle="bold"
                    android:textColor="@color/text_heading" />
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
                    android:textColor="@color/white"
                    android:textSize="@dimen/type_caption"
                    android:textAllCaps="false"
                    app:icon="@drawable/ic_user_chevron"
                    app:iconTint="@color/white"
                    app:iconGravity="textEnd"
                    app:iconPadding="2dp"
                    app:iconSize="12dp" />
            </LinearLayout>
        </LinearLayout>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

#### `app/src/main/res/layout/sample_notification_list.xml`

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
        android:tint="@color/white" />
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
            android:textSize="@dimen/type_body"
            android:textStyle="bold"
            android:textColor="@color/text_heading" />
        <TextView
            android:id="@+id/notification_message"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="2dp"
            android:text="@string/dashboard_notification_message"
            android:textSize="@dimen/type_label"
            android:textColor="@color/text_subheading" />
        <TextView
            android:id="@+id/notification_time"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:text="@string/dashboard_notification_time"
            android:textSize="@dimen/type_caption"
            android:textColor="@color/primary_dark" />
    </LinearLayout>
    <ImageView
        android:id="@+id/notification_remove"
        android:layout_width="32dp"
        android:layout_height="32dp"
        android:layout_marginStart="8dp"
        android:background="@drawable/user_detail_background"
        android:padding="9dp"
        android:src="@drawable/ic_notification_close"
        android:tint="@color/text_body" />
</LinearLayout>
```

#### `app/src/main/res/layout/sample_person_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginVertical="7dp"
    app:cardBackgroundColor="@color/white"
    app:cardCornerRadius="20dp"
    app:cardElevation="2dp"
    app:strokeColor="@color/search_border"
    app:strokeWidth="1dp">
    <LinearLayout
        android:background="@color/white"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center_vertical"
            android:orientation="horizontal">
            <FrameLayout
                android:layout_width="50dp"
                android:layout_height="50dp">
                <TextView
                    android:id="@+id/user_initials"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:background="@drawable/user_avatar_background"
                    android:gravity="center"
                    android:text="@string/sample_user_initials"
                    android:textColor="@color/white"
                    android:textSize="@dimen/type_card_heading"
                    android:textStyle="bold" />
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
                android:layout_marginStart="12dp"
                android:layout_weight="1"
                android:orientation="vertical">
                <TextView
                    android:id="@+id/user_name"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:ellipsize="end"
                    android:maxLines="1"
                    android:text="@string/sample_user_name"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_person_name"
                    android:textStyle="bold" />
                <TextView
                    android:id="@+id/user_email"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="3dp"
                    android:ellipsize="end"
                    android:maxLines="1"
                    android:text="@string/sample_user_email"
                    android:textColor="@color/text_subheading"
                    android:textSize="@dimen/type_label" />
            </LinearLayout>
            <LinearLayout
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:gravity="end"
                android:orientation="vertical">
                <TextView
                    android:id="@+id/user_status"
                    style="@style/green_badge"
                    android:text="@string/user_active" />
                <TextView
                    android:id="@+id/user_joined"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="5dp"
                    android:text="@string/sample_user_joined"
                    android:textColor="@color/text_subheading"
                    android:textSize="@dimen/type_caption" />
            </LinearLayout>
        </LinearLayout>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:orientation="horizontal">
            <TextView
                android:id="@+id/user_role"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:background="@drawable/person_tag_background"
                android:paddingHorizontal="12dp"
                android:paddingVertical="7dp"
                android:text="@string/sample_user_role"
                android:textColor="@color/primary_dark"
                android:textSize="@dimen/type_label" />
            <TextView
                android:id="@+id/user_specialty"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginStart="8dp"
                android:background="@drawable/person_tag_secondary_background"
                android:paddingHorizontal="12dp"
                android:paddingVertical="7dp"
                android:text="@string/sample_user_specialty"
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_label" />
        </LinearLayout>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="14dp"
            android:background="@drawable/person_facts_background"
            android:orientation="horizontal"
            android:paddingHorizontal="10dp"
            android:paddingVertical="12dp">
            <LinearLayout style="@style/PersonFactColumn">
                <TextView style="@style/PersonFactLabel" android:text="@string/person_age_label" />
                <TextView android:id="@+id/user_age" style="@style/PersonFactValue" android:text="@string/sample_user_age" />
            </LinearLayout>
            <LinearLayout style="@style/PersonFactColumn">
                <TextView style="@style/PersonFactLabel" android:text="@string/person_gender_label" />
                <TextView android:id="@+id/user_gender" style="@style/PersonFactValue" android:text="@string/person_gender_sample" />
            </LinearLayout>
            <LinearLayout style="@style/PersonFactColumn">
                <TextView style="@style/PersonFactLabel" android:text="@string/person_location_label" />
                <TextView android:id="@+id/user_location" style="@style/PersonFactValue" android:text="@string/sample_user_location" />
            </LinearLayout>
            <LinearLayout style="@style/PersonFactColumn">
                <TextView style="@style/PersonFactLabel" android:text="@string/person_credits_label" />
                <TextView android:id="@+id/user_credits" style="@style/PersonFactValue" android:text="@string/sample_user_credits" />
            </LinearLayout>
        </LinearLayout>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="14dp"
            android:gravity="center_vertical"
            android:orientation="horizontal">
            <TextView
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="@string/person_quick_actions"
                android:textColor="@color/text_subheading"
                android:textSize="@dimen/type_label" />
            <androidx.appcompat.widget.AppCompatButton
                android:id="@+id/user_edit"
                android:layout_width="82dp"
                android:layout_height="38dp"
                android:background="@drawable/admin_outline_background"
                android:elevation="2dp"
                android:text="@string/user_edit"
                android:textAllCaps="false"
                android:textColor="@color/primary_dark"
                android:textSize="@dimen/type_subheading" />
            <androidx.appcompat.widget.AppCompatButton
                android:id="@+id/user_delete"
                android:layout_width="82dp"
                android:layout_height="38dp"
                android:layout_marginStart="8dp"
                android:background="@drawable/admin_outline_background"
                android:elevation="2dp"
                android:text="@string/user_delete"
                android:textAllCaps="false"
                android:textColor="@color/destructive"
                android:textSize="@dimen/type_subheading" />
        </LinearLayout>
    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

#### `app/src/main/res/layout/toast_banner.xml`

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
    android:accessibilityLiveRegion="polite"
    tools:background="@drawable/modal_card">
    <TextView
        android:id="@+id/toast_icon"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:gravity="center"
        android:includeFontPadding="false"
        android:textSize="@dimen/type_large_heading"
        android:textStyle="bold"
        android:textColor="@color/white"
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
            android:textSize="@dimen/type_body_large"
            android:textStyle="bold"
            android:textColor="@color/text_heading"
            tools:text="Changes saved" />
        <TextView
            android:id="@+id/toast_message"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="4dp"
            android:textSize="@dimen/type_body"
            android:textColor="@color/text_body"
            tools:text="Your changes have been saved successfully." />
    </LinearLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/view_empty_list.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="16dp"
    android:background="@drawable/admin_outline_background"
    android:gravity="center"
    android:orientation="vertical"
    android:paddingHorizontal="24dp"
    android:paddingVertical="38dp">
    <FrameLayout
        android:layout_width="76dp"
        android:layout_height="76dp"
        android:background="@drawable/user_avatar_background">
        <ImageView
            android:id="@+id/empty_list_icon"
            android:layout_width="36dp"
            android:layout_height="36dp"
            android:layout_gravity="center"
            android:src="@drawable/ic_admin_box"
            android:tint="@color/primary_dark" />
    </FrameLayout>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="18dp"
        android:fontFamily="@font/arial_rounded_mt_bold"
        android:text="@string/empty_list_title"
        android:textColor="@color/text_heading"
        android:textSize="@dimen/type_card_heading_large" />
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="6dp"
        android:gravity="center"
        android:text="@string/empty_list_message"
        android:textColor="@color/text_subheading"
        android:textSize="@dimen/type_subheading" />
</LinearLayout>
```

#### `app/src/main/res/layout/activity_sample_login.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    android:fillViewport="true">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="390dp"
            android:background="@drawable/auth_hero_gradient">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                android:paddingHorizontal="28dp"
                android:paddingTop="24dp"
                android:paddingBottom="108dp">
                <androidx.appcompat.widget.AppCompatImageButton
                    android:id="@+id/auth_back"
                    android:layout_width="40dp"
                    android:layout_height="40dp"
                    android:background="@drawable/auth_back_background"
                    android:contentDescription="@string/auth_back"
                    android:padding="10dp"
                    android:src="@drawable/ic_list_back"
                    android:tint="@color/white" />
                <Space
                    android:layout_width="1dp"
                    android:layout_height="0dp"
                    android:layout_weight="1" />
                <TextView
                    android:id="@+id/auth_hero_title"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:fontFamily="@font/arial_rounded_mt_bold"
                    android:textColor="@color/white"
                    android:textSize="@dimen/type_auth_hero"
                    android:text="@string/auth_login_title" />
                <TextView
                    android:id="@+id/auth_hero_subtitle"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:textColor="@color/primary_light"
                    android:textSize="@dimen/type_body_large"
                    android:text="@string/auth_login_subtitle" />
            </LinearLayout>
        </FrameLayout>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="-34dp"
            android:background="@drawable/auth_sheet_background"
            android:orientation="vertical"
            android:paddingHorizontal="24dp"
            android:paddingTop="34dp"
            android:paddingBottom="42dp">
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="24dp"
                android:text="@string/auth_email"
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_subheading" />
            <EditText
                android:id="@+id/auth_email"
                android:layout_width="match_parent"
                android:layout_height="58dp"
                android:layout_marginTop="8dp"
                android:background="@drawable/auth_field_background"
                android:hint="@string/auth_email_hint"
                android:inputType="textEmailAddress"
                android:paddingHorizontal="20dp"
                android:singleLine="true"
                android:textColor="@color/field_text"
                android:textColorHint="@color/field_hint"
                android:textSize="@dimen/type_body"
                android:drawablePadding="14dp"
                android:drawableStart="@drawable/ic_user_email" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="20dp"
                android:text="@string/auth_password"
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_subheading" />
            <EditText
                android:id="@+id/auth_password"
                android:layout_width="match_parent"
                android:layout_height="58dp"
                android:layout_marginTop="8dp"
                android:background="@drawable/auth_field_background"
                android:hint="@string/auth_password_hint"
                android:inputType="textPassword"
                android:paddingHorizontal="20dp"
                android:singleLine="true"
                android:textColor="@color/field_text"
                android:textColorHint="@color/field_hint"
                android:textSize="@dimen/type_body"
                android:drawablePadding="14dp"
                android:drawableStart="@drawable/ic_auth_lock"
                android:drawableEnd="@drawable/ic_auth_eye" />
            <LinearLayout
                android:id="@+id/auth_login_options"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="18dp"
                android:gravity="end"
                android:orientation="horizontal">
                <TextView
                    android:id="@+id/auth_forgot"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:clickable="true"
                    android:focusable="true"
                    android:text="@string/auth_forgot"
                    android:textColor="@color/primary_dark"
                    android:textSize="@dimen/type_label" />
            </LinearLayout>
            <androidx.appcompat.widget.AppCompatButton
                android:id="@+id/auth_submit"
                android:layout_width="match_parent"
                android:layout_height="60dp"
                android:layout_marginTop="28dp"
                android:background="@drawable/auth_button_background"
                android:backgroundTint="@null"
                android:text="@string/auth_login_action"
                android:textAllCaps="false"
                android:textColor="@color/white"
                android:textSize="@dimen/type_button"
                android:drawableEnd="@drawable/ic_auth_arrow"
                android:paddingStart="24dp"
                android:paddingEnd="24dp" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="14dp"
                android:gravity="center"
                android:text="@string/auth_demo_credentials"
                android:textColor="@color/text_subheading"
                android:textSize="@dimen/type_caption" />
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="30dp"
                android:gravity="center"
                android:orientation="horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="@string/auth_no_account"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_body" />
                <TextView
                    android:id="@+id/auth_switch_link"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="5dp"
                    android:clickable="true"
                    android:focusable="true"
                    android:text="@string/auth_signup_link"
                    android:textColor="@color/primary_dark"
                    android:textSize="@dimen/type_body" />
            </LinearLayout>
        </LinearLayout>
    </LinearLayout>
</ScrollView>
```

#### `app/src/main/res/layout/activity_sample_register.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    android:fillViewport="true">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="390dp"
            android:background="@drawable/auth_hero_gradient">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                android:paddingHorizontal="28dp"
                android:paddingTop="24dp"
                android:paddingBottom="108dp">
                <androidx.appcompat.widget.AppCompatImageButton
                    android:id="@+id/auth_back"
                    android:layout_width="40dp"
                    android:layout_height="40dp"
                    android:background="@drawable/auth_back_background"
                    android:contentDescription="@string/auth_back"
                    android:padding="10dp"
                    android:src="@drawable/ic_list_back"
                    android:tint="@color/white" />
                <Space
                    android:layout_width="1dp"
                    android:layout_height="0dp"
                    android:layout_weight="1" />
                <TextView
                    android:id="@+id/auth_hero_title"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:fontFamily="@font/arial_rounded_mt_bold"
                    android:textColor="@color/white"
                    android:textSize="@dimen/type_auth_hero"
                    android:text="@string/auth_signup_title" />
                <TextView
                    android:id="@+id/auth_hero_subtitle"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="10dp"
                    android:textColor="@color/primary_light"
                    android:textSize="@dimen/type_body_large"
                    android:text="@string/auth_signup_subtitle" />
            </LinearLayout>
        </FrameLayout>
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="-34dp"
            android:background="@drawable/auth_sheet_background"
            android:orientation="vertical"
            android:paddingHorizontal="24dp"
            android:paddingTop="34dp"
            android:paddingBottom="42dp">
            <LinearLayout
                android:id="@+id/auth_register_fields"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:text="@string/auth_profile_photo_label"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_subheading" />
                <FrameLayout
                    android:id="@+id/auth_image_picker"
                    android:layout_width="match_parent"
                    android:layout_height="128dp"
                    android:layout_marginTop="8dp"
                    android:background="@drawable/form_upload_background"
                    android:clickable="true"
                    android:focusable="true">
                    <ImageView
                        android:id="@+id/auth_image_preview"
                        android:layout_width="match_parent"
                        android:layout_height="match_parent"
                        android:contentDescription="@string/auth_upload_title"
                        android:scaleType="fitCenter"
                        android:visibility="gone" />
                    <LinearLayout
                        android:id="@+id/auth_image_prompt"
                        android:layout_width="match_parent"
                        android:layout_height="match_parent"
                        android:gravity="center"
                        android:orientation="vertical">
                        <ImageView
                            android:layout_width="28dp"
                            android:layout_height="28dp"
                            android:contentDescription="@null"
                            android:src="@drawable/ic_form_upload"
                            android:tint="@color/primary_dark" />
                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginTop="6dp"
                            android:text="@string/auth_upload_title"
                            android:textColor="@color/text_heading"
                            android:textSize="@dimen/type_body" />
                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginTop="2dp"
                            android:text="@string/auth_upload_subtitle"
                            android:textColor="@color/text_subheading"
                            android:textSize="@dimen/type_label" />
                    </LinearLayout>
                </FrameLayout>
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="20dp"
                    android:orientation="horizontal">
                    <LinearLayout
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:orientation="vertical">
                        <TextView
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:text="@string/auth_first_name"
                            android:textColor="@color/text_heading"
                            android:textSize="@dimen/type_subheading" />
                        <EditText
                            android:id="@+id/auth_first_name"
                            android:layout_width="match_parent"
                            android:layout_height="58dp"
                            android:layout_marginTop="8dp"
                            android:background="@drawable/auth_field_background"
                            android:hint="@string/auth_first_name_hint"
                            android:inputType="textPersonName"
                            android:paddingHorizontal="20dp"
                            android:singleLine="true"
                            android:textColor="@color/field_text"
                            android:textColorHint="@color/field_hint"
                            android:textSize="@dimen/type_body"
                            android:drawablePadding="14dp" />
                    </LinearLayout>
                    <LinearLayout
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_marginStart="12dp"
                        android:layout_weight="1"
                        android:orientation="vertical">
                        <TextView
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:text="@string/auth_last_name"
                            android:textColor="@color/text_heading"
                            android:textSize="@dimen/type_subheading" />
                        <EditText
                            android:id="@+id/auth_last_name"
                            android:layout_width="match_parent"
                            android:layout_height="58dp"
                            android:layout_marginTop="8dp"
                            android:background="@drawable/auth_field_background"
                            android:hint="@string/auth_last_name_hint"
                            android:inputType="textPersonName"
                            android:paddingHorizontal="20dp"
                            android:singleLine="true"
                            android:textColor="@color/field_text"
                            android:textColorHint="@color/field_hint"
                            android:textSize="@dimen/type_body"
                            android:drawablePadding="14dp" />
                    </LinearLayout>
                </LinearLayout>
            </LinearLayout>
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="24dp"
                android:text="@string/auth_email"
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_subheading" />
            <EditText
                android:id="@+id/auth_email"
                android:layout_width="match_parent"
                android:layout_height="58dp"
                android:layout_marginTop="8dp"
                android:background="@drawable/auth_field_background"
                android:hint="@string/auth_email_hint"
                android:inputType="textEmailAddress"
                android:paddingHorizontal="20dp"
                android:singleLine="true"
                android:textColor="@color/field_text"
                android:textColorHint="@color/field_hint"
                android:textSize="@dimen/type_body"
                android:drawablePadding="14dp"
                android:drawableStart="@drawable/ic_user_email" />
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="20dp"
                android:text="@string/auth_password"
                android:textColor="@color/text_heading"
                android:textSize="@dimen/type_subheading" />
            <EditText
                android:id="@+id/auth_password"
                android:layout_width="match_parent"
                android:layout_height="58dp"
                android:layout_marginTop="8dp"
                android:background="@drawable/auth_field_background"
                android:hint="@string/auth_password_hint"
                android:inputType="textPassword"
                android:paddingHorizontal="20dp"
                android:singleLine="true"
                android:textColor="@color/field_text"
                android:textColorHint="@color/field_hint"
                android:textSize="@dimen/type_body"
                android:drawablePadding="14dp"
                android:drawableStart="@drawable/ic_auth_lock"
                android:drawableEnd="@drawable/ic_auth_eye" />
            <LinearLayout
                android:id="@+id/auth_confirm_field"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="20dp"
                android:orientation="vertical">
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="@string/auth_confirm_password"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_subheading" />
                <EditText
                    android:id="@+id/auth_confirm_password"
                    android:layout_width="match_parent"
                    android:layout_height="58dp"
                    android:layout_marginTop="8dp"
                    android:background="@drawable/auth_field_background"
                    android:hint="@string/auth_confirm_hint"
                    android:inputType="textPassword"
                    android:paddingHorizontal="20dp"
                    android:singleLine="true"
                    android:textColor="@color/field_text"
                    android:textColorHint="@color/field_hint"
                    android:textSize="@dimen/type_body"
                    android:drawablePadding="14dp"
                    android:drawableStart="@drawable/ic_auth_lock" />
            </LinearLayout>
            <androidx.appcompat.widget.AppCompatButton
                android:id="@+id/auth_submit"
                android:layout_width="match_parent"
                android:layout_height="60dp"
                android:layout_marginTop="28dp"
                android:background="@drawable/auth_button_background"
                android:backgroundTint="@null"
                android:text="@string/auth_register_action"
                android:textAllCaps="false"
                android:textColor="@color/white"
                android:textSize="@dimen/type_button"
                android:drawableEnd="@drawable/ic_auth_arrow"
                android:paddingStart="24dp"
                android:paddingEnd="24dp" />
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="30dp"
                android:gravity="center"
                android:orientation="horizontal">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="@string/auth_have_account"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_body" />
                <TextView
                    android:id="@+id/auth_switch_link"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="5dp"
                    android:clickable="true"
                    android:focusable="true"
                    android:text="@string/auth_signin_link"
                    android:textColor="@color/primary_dark"
                    android:textSize="@dimen/type_body" />
            </LinearLayout>
        </LinearLayout>
    </LinearLayout>
</ScrollView>
```

#### `app/src/main/res/layout/activity_admin_profile.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/activity_show_background"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="64dp"
        android:background="@color/white"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        android:paddingHorizontal="16dp">
        <androidx.appcompat.widget.AppCompatImageButton
            android:id="@+id/profile_back"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:contentDescription="@string/auth_back"
            android:padding="8dp"
            android:src="@drawable/ic_list_back" />
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="@string/admin_profile_title"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_page_heading" />
        <Space
            android:layout_width="40dp"
            android:layout_height="40dp" />
    </LinearLayout>
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">
        <com.google.android.material.card.MaterialCardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="16dp"
            app:cardBackgroundColor="@color/white"
            app:cardCornerRadius="26dp"
            app:cardElevation="3dp">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
                <FrameLayout
                    android:layout_width="match_parent"
                    android:layout_height="196dp">
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="152dp"
                        android:background="@drawable/profile_cover_background" />
                    <TextView
                        android:id="@+id/profile_avatar"
                        android:layout_width="80dp"
                        android:layout_height="80dp"
                        android:layout_gravity="top|start"
                        android:layout_marginStart="24dp"
                        android:layout_marginTop="112dp"
                        android:background="@drawable/user_avatar_background"
                        android:elevation="4dp"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:gravity="center"
                        android:text="@string/admin_initials"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_metric" />
                </FrameLayout>
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:paddingHorizontal="24dp"
                    android:paddingBottom="24dp">
                    <TextView
                        android:id="@+id/profile_display_name"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/admin_profile_name"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_page_heading" />
                    <TextView
                        android:id="@+id/profile_display_email"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:text="@string/admin_profile_email"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="24dp"
                        android:background="@drawable/person_facts_background"
                        android:orientation="horizontal"
                        android:paddingHorizontal="10dp"
                        android:paddingVertical="14dp">
                        <LinearLayout
                            style="@style/PersonFactColumn">
                            <TextView
                                style="@style/PersonFactLabel"
                                android:text="@string/profile_role_label" />
                            <TextView
                                style="@style/PersonFactValue"
                                android:text="@string/admin_profile_role" />
                        </LinearLayout>
                        <LinearLayout
                            style="@style/PersonFactColumn">
                            <TextView
                                style="@style/PersonFactLabel"
                                android:text="@string/profile_access_label" />
                            <TextView
                                style="@style/PersonFactValue"
                                android:text="@string/admin_profile_access" />
                        </LinearLayout>
                        <LinearLayout
                            style="@style/PersonFactColumn">
                            <TextView
                                style="@style/PersonFactLabel"
                                android:text="@string/profile_status_label" />
                            <TextView
                                style="@style/PersonFactValue"
                                android:text="@string/admin_profile_status" />
                        </LinearLayout>
                    </LinearLayout>
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="1dp"
                        android:layout_marginTop="28dp"
                        android:background="@color/page_divider" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="26dp"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/profile_public_heading"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_card_heading" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:text="@string/profile_public_hint"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="22dp"
                        android:text="@string/profile_name_label"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_label" />
                    <EditText
                        android:id="@+id/profile_name_input"
                        android:layout_width="match_parent"
                        android:layout_height="54dp"
                        android:layout_marginTop="8dp"
                        android:background="@drawable/form_field_background"
                        android:inputType="textPersonName"
                        android:paddingHorizontal="14dp"
                        android:singleLine="true"
                        android:textColor="@color/field_text"
                        android:textSize="@dimen/type_body" />
                    <TextView
                        android:id="@+id/profile_name_error"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="5dp"
                        android:text="@string/profile_name_error"
                        android:textColor="@color/field_error"
                        android:textSize="@dimen/type_caption"
                        android:visibility="gone" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="18dp"
                        android:text="@string/profile_email_label"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_label" />
                    <EditText
                        android:id="@+id/profile_email_input"
                        android:layout_width="match_parent"
                        android:layout_height="54dp"
                        android:layout_marginTop="8dp"
                        android:background="@drawable/form_field_background"
                        android:inputType="textEmailAddress"
                        android:paddingHorizontal="14dp"
                        android:singleLine="true"
                        android:textColor="@color/field_text"
                        android:textSize="@dimen/type_body" />
                    <TextView
                        android:id="@+id/profile_email_error"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="5dp"
                        android:text="@string/profile_email_error"
                        android:textColor="@color/field_error"
                        android:textSize="@dimen/type_caption"
                        android:visibility="gone" />
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="1dp"
                        android:layout_marginTop="28dp"
                        android:background="@color/page_divider" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="26dp"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/profile_preferences_heading"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_card_heading" />
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="20dp"
                        android:gravity="center_vertical"
                        android:orientation="horizontal">
                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:text="@string/profile_notifications_label"
                                android:textColor="@color/text_heading"
                                android:textSize="@dimen/type_body" />
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="4dp"
                                android:text="@string/profile_notifications_hint"
                                android:textColor="@color/text_subheading"
                                android:textSize="@dimen/type_subheading" />
                        </LinearLayout>
                        <com.google.android.material.switchmaterial.SwitchMaterial
                            android:id="@+id/profile_notifications_switch"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginStart="12dp"
                            app:thumbTint="@color/primary_dark"
                            app:trackTint="@color/primary_light" />
                    </LinearLayout>
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="26dp"
                        android:gravity="center_vertical"
                        android:orientation="horizontal">
                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:text="@string/profile_logout_label"
                                android:textColor="@color/text_heading"
                                android:textSize="@dimen/type_body" />
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="4dp"
                                android:text="@string/profile_logout_hint"
                                android:textColor="@color/text_subheading"
                                android:textSize="@dimen/type_subheading" />
                        </LinearLayout>
                        <androidx.appcompat.widget.AppCompatButton
                            android:id="@+id/profile_logout"
                            android:layout_width="90dp"
                            android:layout_height="40dp"
                            android:layout_marginStart="10dp"
                            android:background="@drawable/admin_outline_background"
                            android:text="@string/profile_logout_label"
                            android:textAllCaps="false"
                            android:textColor="@color/destructive"
                            android:textSize="@dimen/type_label" />
                    </LinearLayout>
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="1dp"
                        android:layout_marginTop="28dp"
                        android:background="@color/page_divider" />
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="24dp"
                        android:gravity="end"
                        android:orientation="horizontal">
                        <androidx.appcompat.widget.AppCompatButton
                            android:id="@+id/profile_cancel"
                            android:layout_width="100dp"
                            android:layout_height="48dp"
                            android:background="@drawable/admin_outline_background"
                            android:text="@string/profile_cancel"
                            android:textAllCaps="false"
                            android:textColor="@color/text_heading"
                            android:textSize="@dimen/type_body" />
                        <androidx.appcompat.widget.AppCompatButton
                            android:id="@+id/profile_save"
                            android:layout_width="140dp"
                            android:layout_height="48dp"
                            android:layout_marginStart="10dp"
                            android:background="@drawable/form_action_solid"
                            android:text="@string/profile_save"
                            android:textAllCaps="false"
                            android:textColor="@color/white"
                            android:textSize="@dimen/type_body" />
                    </LinearLayout>
                </LinearLayout>
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>
    </ScrollView>
</LinearLayout>
```

#### `app/src/main/res/layout/popup_admin_account.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="188dp"
    android:layout_height="wrap_content"
    android:background="@drawable/admin_account_menu_background"
    android:orientation="vertical"
    android:padding="8dp">
    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/account_profile"
        android:layout_width="match_parent"
        android:layout_height="46dp"
        android:background="@android:color/transparent"
        android:backgroundTint="@null"
        android:gravity="center_vertical|start"
        android:minWidth="0dp"
        android:paddingHorizontal="14dp"
        android:text="@string/admin_profile_action"
        android:textAllCaps="false"
        android:textColor="@color/text_heading"
        android:textSize="@dimen/type_body" />
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:layout_marginHorizontal="10dp"
        android:background="@color/page_divider" />
    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/account_logout"
        android:layout_width="match_parent"
        android:layout_height="46dp"
        android:background="@android:color/transparent"
        android:backgroundTint="@null"
        android:gravity="center_vertical|start"
        android:minWidth="0dp"
        android:paddingHorizontal="14dp"
        android:text="@string/admin_logout_action"
        android:textAllCaps="false"
        android:textColor="@color/destructive"
        android:textSize="@dimen/type_body" />
</LinearLayout>
```

#### `app/src/main/res/layout/activity_personal_dashboard.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white"
    android:orientation="vertical"
    tools:context=".PersonalDashboardActivity">
    <LinearLayout
        android:id="@+id/personal_header"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/dashboard_header_background"
        android:elevation="8dp"
        android:orientation="vertical"
        android:paddingHorizontal="22dp"
        android:paddingTop="22dp"
        android:paddingBottom="20dp">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center_vertical"
            android:orientation="horizontal">
            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:orientation="vertical">
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:fontFamily="@font/arial_rounded_mt_bold"
                    android:text="@string/personal_dashboard_title"
                    android:textColor="@color/text_heading"
                    android:textSize="@dimen/type_page_heading" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="4dp"
                    android:text="@string/personal_dashboard_subtitle"
                    android:textColor="@color/text_subheading"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
            <FrameLayout
                android:id="@+id/personal_notifications_button"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:layout_marginStart="8dp"
                android:clickable="true"
                android:contentDescription="@string/personal_dashboard_notifications"
                android:focusable="true"
                android:foreground="?attr/selectableItemBackgroundBorderless">
                <ImageView
                    android:id="@+id/personal_bell_icon"
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:contentDescription="@null"
                    android:padding="8dp"
                    android:src="@drawable/ic_dashboard_bell"
                    android:tint="@color/dashboard_bell_tint" />
                <View
                    android:id="@+id/personal_notification_dot"
                    android:layout_width="8dp"
                    android:layout_height="8dp"
                    android:layout_gravity="top|end"
                    android:layout_margin="2dp"
                    android:background="@drawable/user_avatar_background"
                    android:backgroundTint="@color/destructive" />
            </FrameLayout>
            <TextView
                android:id="@+id/personal_avatar_button"
                android:layout_width="40dp"
                android:layout_height="40dp"
                android:layout_marginStart="8dp"
                android:background="@drawable/admin_avatar_background"
                android:clickable="true"
                android:contentDescription="@string/personal_dashboard_account"
                android:focusable="true"
                android:fontFamily="@font/arial_rounded_mt_bold"
                android:foreground="?attr/selectableItemBackgroundBorderless"
                android:gravity="center"
                android:text="@string/personal_dashboard_initials"
                android:textColor="@color/primary_dark"
                android:textSize="@dimen/type_body" />
        </LinearLayout>
        <LinearLayout
            android:id="@+id/personal_search_trigger"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:layout_marginTop="18dp"
            android:background="@drawable/menu_search_background"
            android:clickable="true"
            android:focusable="true"
            android:foreground="?attr/selectableItemBackground"
            android:gravity="center_vertical"
            android:orientation="horizontal">
            <ImageView
                android:layout_width="44dp"
                android:layout_height="match_parent"
                android:contentDescription="@null"
                android:padding="12dp"
                android:src="@drawable/ic_list_search"
                android:tint="@color/text_body" />
            <TextView
                android:id="@+id/personal_search"
                android:layout_width="0dp"
                android:layout_height="match_parent"
                android:layout_weight="1"
                android:background="@android:color/transparent"
                android:gravity="center_vertical"
                android:paddingEnd="18dp"
                android:text="@string/personal_dashboard_search"
                android:textColor="@color/search_hint"
                android:textSize="@dimen/type_body" />
        </LinearLayout>
    </LinearLayout>
    <ScrollView
        android:id="@+id/personal_content"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:paddingHorizontal="22dp"
            android:paddingTop="20dp"
            android:paddingBottom="24dp">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <include
                        android:id="@+id/seat_three"
                        layout="@layout/view_personal_seat_tile"
                        android:layout_width="match_parent"
                        android:layout_height="224dp" />
                    <include
                        android:id="@+id/seat_six"
                        layout="@layout/view_personal_seat_tile"
                        android:layout_width="match_parent"
                        android:layout_height="172dp"
                        android:layout_marginTop="10dp" />
                </LinearLayout>
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="10dp"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <include
                        android:id="@+id/seat_five"
                        layout="@layout/view_personal_seat_tile"
                        android:layout_width="match_parent"
                        android:layout_height="172dp" />
                    <include
                        android:id="@+id/seat_eight"
                        layout="@layout/view_personal_seat_tile"
                        android:layout_width="match_parent"
                        android:layout_height="224dp"
                        android:layout_marginTop="10dp" />
                </LinearLayout>
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="24dp"
                android:gravity="center_vertical"
                android:orientation="horizontal">
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/personal_featured_title"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_section_heading" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="3dp"
                        android:text="@string/personal_featured_subtitle"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
                <TextView
                    android:id="@+id/personal_featured_count"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:background="@drawable/menu_search_background"
                    android:gravity="center"
                    android:minWidth="42dp"
                    android:paddingHorizontal="12dp"
                    android:paddingVertical="7dp"
                    android:text="@string/admin_zero_items"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/personal_featured_items"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:clipToPadding="false"
                android:nestedScrollingEnabled="false"
                android:scrollbars="none"
                tools:itemCount="3"
                tools:listitem="@layout/sample_item_card_list" />
            <include
                android:id="@+id/personal_featured_empty"
                layout="@layout/view_empty_list"
                android:visibility="gone" />
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="22dp"
                android:gravity="center_vertical"
                android:orientation="horizontal">
                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/personal_activities_title"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_section_heading" />
                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="3dp"
                        android:text="@string/personal_activities_subtitle"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                </LinearLayout>
                <TextView
                    android:id="@+id/personal_activity_count"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:background="@drawable/menu_search_background"
                    android:gravity="center"
                    android:minWidth="42dp"
                    android:paddingHorizontal="12dp"
                    android:paddingVertical="7dp"
                    android:text="@string/admin_zero_items"
                    android:textColor="@color/text_body"
                    android:textSize="@dimen/type_subheading" />
            </LinearLayout>
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="48dp"
                android:layout_marginTop="14dp"
                android:background="@drawable/admin_outline_background"
                android:gravity="center_vertical"
                android:orientation="horizontal">
                <EditText
                    android:id="@+id/personal_activities_search"
                    android:layout_width="0dp"
                    android:layout_height="match_parent"
                    android:layout_weight="1"
                    android:background="@android:color/transparent"
                    android:hint="@string/personal_activities_search_hint"
                    android:inputType="text"
                    android:paddingStart="18dp"
                    android:paddingEnd="4dp"
                    android:singleLine="true"
                    android:textColor="@color/search_text"
                    android:textColorHint="@color/search_hint"
                    android:textSize="@dimen/type_body" />
                <ImageView
                    android:layout_width="42dp"
                    android:layout_height="match_parent"
                    android:contentDescription="@null"
                    android:padding="12dp"
                    android:src="@drawable/ic_list_search"
                    android:tint="@color/text_heading" />
            </LinearLayout>
            <com.google.android.material.chip.ChipGroup
                android:id="@+id/personal_activity_filters"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                app:chipSpacingHorizontal="8dp"
                app:chipSpacingVertical="8dp"
                app:selectionRequired="true"
                app:singleLine="false"
                app:singleSelection="true" />
            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/personal_activity_list"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:nestedScrollingEnabled="false"
                tools:itemCount="3"
                tools:listitem="@layout/sample_activity_card_list" />
            <include
                android:id="@+id/personal_activities_empty"
                layout="@layout/view_empty_list"
                android:visibility="gone" />
        </LinearLayout>
    </ScrollView>
</LinearLayout>
```

#### `app/src/main/res/layout/bottom_sheet_personal_search.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/dashboard_sheet_background"
    android:orientation="vertical"
    android:paddingHorizontal="16dp"
    android:paddingBottom="20dp">
    <View
        android:layout_width="36dp"
        android:layout_height="4dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="10dp"
        android:background="@drawable/menu_search_background"
        android:backgroundTint="@color/primary_light" />
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="56dp"
        android:layout_marginTop="12dp"
        android:gravity="center_vertical"
        android:orientation="horizontal">
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:fontFamily="@font/arial_rounded_mt_bold"
            android:text="@string/personal_search_title"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_page_heading" />
        <androidx.appcompat.widget.AppCompatImageButton
            android:id="@+id/personal_search_close"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="@drawable/user_detail_background"
            android:contentDescription="@string/personal_search_close_description"
            android:padding="10dp"
            android:src="@drawable/ic_notification_close" />
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:layout_marginTop="10dp"
        android:background="@drawable/admin_outline_background"
        android:gravity="center_vertical"
        android:orientation="horizontal">
        <EditText
            android:id="@+id/personal_search_input"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:background="@android:color/transparent"
            android:hint="@string/personal_search_hint"
            android:inputType="text"
            android:paddingStart="18dp"
            android:paddingEnd="4dp"
            android:singleLine="true"
            android:textColor="@color/search_text"
            android:textColorHint="@color/search_hint"
            android:textSize="@dimen/type_body" />
        <ImageView
            android:layout_width="42dp"
            android:layout_height="match_parent"
            android:contentDescription="@null"
            android:padding="12dp"
            android:src="@drawable/ic_list_search"
            android:tint="@color/text_heading" />
    </LinearLayout>
    <com.google.android.material.chip.ChipGroup
        android:id="@+id/personal_search_filters"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:layout_marginBottom="8dp"
        app:chipSpacingHorizontal="8dp"
        app:chipSpacingVertical="8dp"
        app:selectionRequired="true"
        app:singleLine="false"
        app:singleSelection="true" />
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/personal_search_results"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:clipToPadding="false"
            android:paddingBottom="16dp"
            app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
            app:spanCount="2"
            tools:itemCount="3"
            tools:listitem="@layout/sample_item_card_list" />
        <include
            android:id="@+id/personal_search_empty"
            layout="@layout/view_empty_list"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:visibility="gone" />
    </FrameLayout>
</LinearLayout>
```

#### `app/src/main/res/layout/activity_personal_profile.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/activity_show_background"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="64dp"
        android:background="@color/white"
        android:gravity="center_vertical"
        android:orientation="horizontal"
        android:paddingHorizontal="16dp">
        <androidx.appcompat.widget.AppCompatImageButton
            android:id="@+id/profile_back"
            android:layout_width="40dp"
            android:layout_height="40dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:contentDescription="@string/auth_back"
            android:padding="8dp"
            android:src="@drawable/ic_list_back" />
        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:text="@string/personal_profile_title"
            android:textColor="@color/text_heading"
            android:textSize="@dimen/type_page_heading" />
        <Space
            android:layout_width="40dp"
            android:layout_height="40dp" />
    </LinearLayout>
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">
        <com.google.android.material.card.MaterialCardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_margin="16dp"
            app:cardBackgroundColor="@color/white"
            app:cardCornerRadius="26dp"
            app:cardElevation="3dp">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">
                <FrameLayout
                    android:layout_width="match_parent"
                    android:layout_height="196dp">
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="152dp"
                        android:background="@drawable/profile_cover_background" />
                    <TextView
                        android:id="@+id/profile_avatar"
                        android:layout_width="80dp"
                        android:layout_height="80dp"
                        android:layout_gravity="top|start"
                        android:layout_marginStart="24dp"
                        android:layout_marginTop="112dp"
                        android:background="@drawable/user_avatar_background"
                        android:elevation="4dp"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:gravity="center"
                        android:text="@string/personal_dashboard_initials"
                        android:textColor="@color/white"
                        android:textSize="@dimen/type_metric" />
                </FrameLayout>
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:paddingHorizontal="24dp"
                    android:paddingBottom="24dp">
                    <TextView
                        android:id="@+id/profile_display_name"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/personal_profile_name"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_page_heading" />
                    <TextView
                        android:id="@+id/profile_display_email"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:text="@string/personal_profile_email"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="24dp"
                        android:background="@drawable/person_facts_background"
                        android:orientation="horizontal"
                        android:paddingHorizontal="10dp"
                        android:paddingVertical="14dp">
                        <LinearLayout
                            style="@style/PersonFactColumn">
                            <TextView
                                style="@style/PersonFactLabel"
                                android:text="@string/profile_role_label" />
                            <TextView
                                style="@style/PersonFactValue"
                                android:text="@string/personal_profile_role" />
                        </LinearLayout>
                        <LinearLayout
                            style="@style/PersonFactColumn">
                            <TextView
                                style="@style/PersonFactLabel"
                                android:text="@string/profile_access_label" />
                            <TextView
                                style="@style/PersonFactValue"
                                android:text="@string/personal_profile_access" />
                        </LinearLayout>
                        <LinearLayout
                            style="@style/PersonFactColumn">
                            <TextView
                                style="@style/PersonFactLabel"
                                android:text="@string/profile_status_label" />
                            <TextView
                                style="@style/PersonFactValue"
                                android:text="@string/personal_profile_status" />
                        </LinearLayout>
                    </LinearLayout>
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="1dp"
                        android:layout_marginTop="28dp"
                        android:background="@color/page_divider" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="26dp"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/profile_public_heading"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_card_heading" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:text="@string/profile_public_hint"
                        android:textColor="@color/text_subheading"
                        android:textSize="@dimen/type_subheading" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="22dp"
                        android:text="@string/profile_name_label"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_label" />
                    <EditText
                        android:id="@+id/profile_name_input"
                        android:layout_width="match_parent"
                        android:layout_height="54dp"
                        android:layout_marginTop="8dp"
                        android:background="@drawable/form_field_background"
                        android:inputType="textPersonName"
                        android:paddingHorizontal="14dp"
                        android:singleLine="true"
                        android:textColor="@color/field_text"
                        android:textSize="@dimen/type_body" />
                    <TextView
                        android:id="@+id/profile_name_error"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="5dp"
                        android:text="@string/profile_name_error"
                        android:textColor="@color/field_error"
                        android:textSize="@dimen/type_caption"
                        android:visibility="gone" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="18dp"
                        android:text="@string/profile_email_label"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_label" />
                    <EditText
                        android:id="@+id/profile_email_input"
                        android:layout_width="match_parent"
                        android:layout_height="54dp"
                        android:layout_marginTop="8dp"
                        android:background="@drawable/form_field_background"
                        android:inputType="textEmailAddress"
                        android:paddingHorizontal="14dp"
                        android:singleLine="true"
                        android:textColor="@color/field_text"
                        android:textSize="@dimen/type_body" />
                    <TextView
                        android:id="@+id/profile_email_error"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="5dp"
                        android:text="@string/profile_email_error"
                        android:textColor="@color/field_error"
                        android:textSize="@dimen/type_caption"
                        android:visibility="gone" />
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="1dp"
                        android:layout_marginTop="28dp"
                        android:background="@color/page_divider" />
                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="26dp"
                        android:fontFamily="@font/arial_rounded_mt_bold"
                        android:text="@string/profile_preferences_heading"
                        android:textColor="@color/text_heading"
                        android:textSize="@dimen/type_card_heading" />
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="20dp"
                        android:gravity="center_vertical"
                        android:orientation="horizontal">
                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:text="@string/profile_notifications_label"
                                android:textColor="@color/text_heading"
                                android:textSize="@dimen/type_body" />
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="4dp"
                                android:text="@string/profile_notifications_hint"
                                android:textColor="@color/text_subheading"
                                android:textSize="@dimen/type_subheading" />
                        </LinearLayout>
                        <com.google.android.material.switchmaterial.SwitchMaterial
                            android:id="@+id/profile_notifications_switch"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginStart="12dp"
                            app:thumbTint="@color/primary_dark"
                            app:trackTint="@color/primary_light" />
                    </LinearLayout>
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="26dp"
                        android:gravity="center_vertical"
                        android:orientation="horizontal">
                        <LinearLayout
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:orientation="vertical">
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:text="@string/profile_logout_label"
                                android:textColor="@color/text_heading"
                                android:textSize="@dimen/type_body" />
                            <TextView
                                android:layout_width="match_parent"
                                android:layout_height="wrap_content"
                                android:layout_marginTop="4dp"
                                android:text="@string/profile_logout_hint"
                                android:textColor="@color/text_subheading"
                                android:textSize="@dimen/type_subheading" />
                        </LinearLayout>
                        <androidx.appcompat.widget.AppCompatButton
                            android:id="@+id/profile_logout"
                            android:layout_width="90dp"
                            android:layout_height="40dp"
                            android:layout_marginStart="10dp"
                            android:background="@drawable/admin_outline_background"
                            android:text="@string/profile_logout_label"
                            android:textAllCaps="false"
                            android:textColor="@color/destructive"
                            android:textSize="@dimen/type_label" />
                    </LinearLayout>
                    <View
                        android:layout_width="match_parent"
                        android:layout_height="1dp"
                        android:layout_marginTop="28dp"
                        android:background="@color/page_divider" />
                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="24dp"
                        android:gravity="end"
                        android:orientation="horizontal">
                        <androidx.appcompat.widget.AppCompatButton
                            android:id="@+id/profile_cancel"
                            android:layout_width="100dp"
                            android:layout_height="48dp"
                            android:background="@drawable/admin_outline_background"
                            android:text="@string/profile_cancel"
                            android:textAllCaps="false"
                            android:textColor="@color/text_heading"
                            android:textSize="@dimen/type_body" />
                        <androidx.appcompat.widget.AppCompatButton
                            android:id="@+id/profile_save"
                            android:layout_width="140dp"
                            android:layout_height="48dp"
                            android:layout_marginStart="10dp"
                            android:background="@drawable/form_action_solid"
                            android:text="@string/profile_save"
                            android:textAllCaps="false"
                            android:textColor="@color/white"
                            android:textSize="@dimen/type_body" />
                    </LinearLayout>
                </LinearLayout>
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>
    </ScrollView>
</LinearLayout>
```

#### `app/src/main/res/layout/view_personal_seat_tile.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/admin_outline_background"
    android:clipToOutline="true">
    <ViewFlipper
        android:id="@+id/seat_flipper"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:autoStart="false"
        android:flipInterval="8000"
        android:inAnimation="@android:anim/fade_in"
        android:outAnimation="@android:anim/fade_out">
        <ImageView
            android:id="@+id/seat_image_first"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:contentDescription="@null"
            android:scaleType="centerCrop" />
        <ImageView
            android:id="@+id/seat_image_second"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:contentDescription="@null"
            android:scaleType="centerCrop" />
    </ViewFlipper>
    <TextView
        android:id="@+id/seat_label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="top|start"
        android:layout_margin="14dp"
        android:fontFamily="@font/arial_rounded_mt_bold"
        android:textColor="@color/text_heading"
        android:textSize="@dimen/type_card_heading" />
</FrameLayout>
```

### res/drawable (71 files)

#### `app/src/main/res/drawable/activity_action_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/white_25">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="270"
                android:startColor="@color/primary"
                android:endColor="@color/success" />
        </shape>
    </item>
</ripple>
```

#### `app/src/main/res/drawable/activity_card_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="20dp" />
    <gradient
        android:angle="225"
        android:startColor="@color/primary_light"
        android:centerColor="@color/white"
        android:endColor="@color/white" />
</shape>
```

#### `app/src/main/res/drawable/activity_icon_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="14dp" />
    <gradient
        android:angle="270"
        android:startColor="@color/primary_light"
        android:endColor="@color/primary_light" />
</shape>
```

#### `app/src/main/res/drawable/activity_show_icon_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <gradient
        android:angle="270"
        android:startColor="@color/primary"
        android:endColor="@color/success" />
</shape>
```

#### `app/src/main/res/drawable/admin_avatar_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="100dp" />
    <solid
        android:color="@color/primary_light" />
</shape>
```

#### `app/src/main/res/drawable/admin_feature_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="24dp" />
    <solid
        android:color="@color/primary_darkest" />
</shape>
```

#### `app/src/main/res/drawable/admin_metric_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="22dp" />
    <gradient
        android:angle="315"
        android:startColor="@color/primary_lime"
        android:endColor="@color/primary_lime" />
</shape>
```

#### `app/src/main/res/drawable/admin_outline_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="22dp" />
    <solid
        android:color="@color/white" />
    <stroke
        android:width="1dp"
        android:color="@color/search_border" />
</shape>
```

#### `app/src/main/res/drawable/admin_transaction_row_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape>
            <solid
                android:color="@color/primary_light" />
        </shape>
    </item>
    <item
        android:bottom="1dp">
        <shape>
            <solid
                android:color="@color/white" />
        </shape>
    </item>
</layer-list>
```

#### `app/src/main/res/drawable/danger_card.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="24dp" />
    <gradient
        android:angle="225"
        android:startColor="@color/white"
        android:centerColor="@color/white"
        android:endColor="@color/white" />
    <stroke
        android:width="1dp"
        android:color="@color/destructive" />
</shape>
```

#### `app/src/main/res/drawable/dashboard_header_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/white" />
    <corners
        android:bottomLeftRadius="24dp"
        android:bottomRightRadius="24dp" />
</shape>
```

#### `app/src/main/res/drawable/dashboard_logo_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="14dp" />
    <gradient
        android:angle="315"
        android:startColor="@color/primary_dark"
        android:endColor="@color/primary_darkest" />
</shape>
```

#### `app/src/main/res/drawable/dashboard_sheet_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:topLeftRadius="24dp"
        android:topRightRadius="24dp" />
    <solid
        android:color="@color/white" />
</shape>
```

#### `app/src/main/res/drawable/dashboard_stat_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="180dp"
    android:height="152dp"
    android:viewportWidth="180"
    android:viewportHeight="152">
    <path
        android:fillColor="@color/primary_light"
        android:pathData="M0,0H180V152H0Z" />
    <path
        android:fillColor="@color/primary_light"
        android:pathData="M65,152C90,115 112,142 136,100C151,76 168,72 180,72V152Z" />
    <path
        android:fillColor="@color/primary_light"
        android:pathData="M94,152C118,111 140,143 180,110V152Z" />
</vector>
```

#### `app/src/main/res/drawable/form_action_solid.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <corners android:radius="8dp" />
    <solid android:color="@color/text_heading" />
</shape>
```

#### `app/src/main/res/drawable/form_field_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <corners android:radius="8dp" />
    <solid android:color="@color/field_background" />
    <stroke
        android:width="1dp"
        android:color="@color/field_border" />
</shape>
```

#### `app/src/main/res/drawable/form_upload_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <corners android:radius="10dp" />
    <solid android:color="@color/white" />
    <stroke
        android:width="1dp"
        android:color="@color/field_upload_border"
        android:dashWidth="5dp"
        android:dashGap="5dp" />
</shape>
```

#### `app/src/main/res/drawable/green_badge.xml`

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
                android:startColor="@color/primary_light"
                android:endColor="@color/primary_light" />
            <stroke
                android:width="1dp"
                android:color="@color/primary_light" />
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
                android:color="@color/primary" />
        </shape>
    </item>
</layer-list>
```

#### `app/src/main/res/drawable/ic_activity_clock.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,2 A10,10 0,1 1,12 22 A10,10 0,1 1,12 2 M12,6 V12 L16,14" />
</vector>
```

#### `app/src/main/res/drawable/ic_activity_more.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="18dp"
    android:height="18dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@color/text_body"
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,3 A1,1 0,1 1,12 5 A1,1 0,1 1,12 3 M12,11 A1,1 0,1 1,12 13 A1,1 0,1 1,12 11 M12,19 A1,1 0,1 1,12 21 A1,1 0,1 1,12 19" />
</vector>
```

#### `app/src/main/res/drawable/ic_activity_people.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M9,3 A4,4 0,1 1,9 11 A4,4 0,1 1,9 3 M2,21 V18 C2,12 16,12 16,18 V21 M17,3 C22,3 22,11 17,11 M19,14 C23,15 23,19 23,21" />
</vector>
```

#### `app/src/main/res/drawable/ic_activity_summary.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="36dp"
    android:height="36dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@android:color/transparent"
        android:strokeColor="@color/white"
        android:strokeWidth="2"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M5,5 H19 Q21,5 21,7 V20 Q21,22 19,22 H5 Q3,22 3,20 V7 Q3,5 5,5 Z M3,10 H21 M8,2 V7 M16,2 V7 M8,15 H16 M8,18 H13" />
</vector>
```

#### `app/src/main/res/drawable/ic_admin_box.xml`

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
        android:strokeColor="@color/white"
        android:strokeWidth="1.8"
        android:strokeLineJoin="round"
        android:pathData="M12,2L21,7v10l-9,5 -9,-5V7z M3,7l9,5 9,-5 M12,12v10" />
</vector>
```

#### `app/src/main/res/drawable/ic_admin_shirt.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@color/primary_darkest"
        android:pathData="M8,3h2c0,1.2 0.8,2 2,2s2,-0.8 2,-2h2l5,3 -2,4 -2,-1v12H7V9l-2,1 -2,-4z" />
</vector>
```

#### `app/src/main/res/drawable/ic_dashboard_bell.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@color/white"
        android:pathData="M12,2c-0.55,0 -1,0.45 -1,1v0.08C7.56,3.56 5,6.5 5,10v4.3l-1.8,2.9c-0.4,0.65 0.07,1.5 0.83,1.5h15.94c0.76,0 1.23,-0.85 0.83,-1.5L19,14.3V10c0,-3.5 -2.56,-6.44 -6,-6.92V3c0,-0.55 -0.45,-1 -1,-1zM9.5,20h5c-0.23,1.16 -1.26,2 -2.5,2s-2.27,-0.84 -2.5,-2z" />
</vector>
```

#### `app/src/main/res/drawable/ic_form_folder.xml`

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
        android:strokeColor="@color/text_heading"
        android:strokeWidth="2"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,7a2,2 0,0 1,2 -2h4l2,2h8a2,2 0,0 1,2 2v9a2,2 0,0 1,-2 2H5a2,2 0,0 1,-2 -2z M8,12h8v5H8z" />
</vector>
```

#### `app/src/main/res/drawable/ic_form_upload.xml`

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
        android:strokeColor="@color/text_subheading"
        android:strokeWidth="2"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M6,17H5a3,3 0,0 1,-0.3 -5.98A7,7 0,0 1,18.4 9.2A4,4 0,0 1,19 17h-1 M12,20V10 M8.5,13.5L12,10l3.5,3.5" />
</vector>
```

#### `app/src/main/res/drawable/ic_item_heart.xml`

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
        android:strokeColor="@color/text_heading"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,21 C8,17 2,13 2,8 C2,2 9,1 12,6 C15,1 22,2 22,8 C22,13 16,17 12,21 Z" />
</vector>
```

#### `app/src/main/res/drawable/ic_item_heart_filled.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="22dp"
    android:height="22dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@color/destructive"
        android:strokeColor="@color/destructive"
        android:strokeWidth="0"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,21 C8,17 2,13 2,8 C2,2 9,1 12,6 C15,1 22,2 22,8 C22,13 16,17 12,21 Z" />
</vector>
```

#### `app/src/main/res/drawable/ic_item_star.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="14dp"
    android:height="14dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
    <path
        android:fillColor="@color/badge_yellow"
        android:strokeColor="@color/badge_yellow"
        android:strokeWidth="0"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,1 L15.4,8.1 L23,9.2 L17.5,14.6 L18.8,22.2 L12,18.6 L5.2,22.2 L6.5,14.6 L1,9.2 L8.6,8.1 Z" />
</vector>
```

#### `app/src/main/res/drawable/ic_launcher_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="@color/primary"
        android:pathData="M0,0h108v108h-108z" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M9,0L9,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M19,0L19,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M29,0L29,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M39,0L39,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M49,0L49,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M59,0L59,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M69,0L69,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M79,0L79,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M89,0L89,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M99,0L99,108"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,9L108,9"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,19L108,19"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,29L108,29"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,39L108,39"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,49L108,49"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,59L108,59"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,69L108,69"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,79L108,79"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,89L108,89"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M0,99L108,99"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M19,29L89,29"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M19,39L89,39"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M19,49L89,49"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M19,59L89,59"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M19,69L89,69"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M19,79L89,79"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M29,19L29,89"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M39,19L39,89"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M49,19L49,89"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M59,19L59,89"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M69,19L69,89"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
    <path
        android:fillColor="@color/transparent"
        android:pathData="M79,19L79,89"
        android:strokeWidth="0.8"
        android:strokeColor="@color/white_20" />
</vector>
```

#### `app/src/main/res/drawable/ic_launcher_foreground.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<vector
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:aapt="http://schemas.android.com/aapt"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:pathData="M31,63.928c0,0 6.4,-11 12.1,-13.1c7.2,-2.6 26,-1.4 26,-1.4l38.1,38.1L107,108.928l-32,-1L31,63.928z">
        <aapt:attr
            name="android:fillColor">
            <gradient
                android:endX="85.84757"
                android:endY="92.4963"
                android:startX="42.9492"
                android:startY="49.59793"
                android:type="linear">
                <item
                    android:color="@color/black_27"
                    android:offset="0.0" />
                <item
                    android:color="@color/transparent"
                    android:offset="1.0" />
            </gradient>
        </aapt:attr>
    </path>
    <path
        android:fillColor="@color/white"
        android:fillType="nonZero"
        android:pathData="M65.3,45.828l3.8,-6.6c0.2,-0.4 0.1,-0.9 -0.3,-1.1c-0.4,-0.2 -0.9,-0.1 -1.1,0.3l-3.9,6.7c-6.3,-2.8 -13.4,-2.8 -19.7,0l-3.9,-6.7c-0.2,-0.4 -0.7,-0.5 -1.1,-0.3C38.8,38.328 38.7,38.828 38.9,39.228l3.8,6.6C36.2,49.428 31.7,56.028 31,63.928h46C76.3,56.028 71.8,49.428 65.3,45.828zM43.4,57.328c-0.8,0 -1.5,-0.5 -1.8,-1.2c-0.3,-0.7 -0.1,-1.5 0.4,-2.1c0.5,-0.5 1.4,-0.7 2.1,-0.4c0.7,0.3 1.2,1 1.2,1.8C45.3,56.528 44.5,57.328 43.4,57.328L43.4,57.328zM64.6,57.328c-0.8,0 -1.5,-0.5 -1.8,-1.2s-0.1,-1.5 0.4,-2.1c0.5,-0.5 1.4,-0.7 2.1,-0.4c0.7,0.3 1.2,1 1.2,1.8C66.5,56.528 65.6,57.328 64.6,57.328L64.6,57.328z"
        android:strokeWidth="1"
        android:strokeColor="@color/transparent" />
</vector>
```

#### `app/src/main/res/drawable/ic_list_back.xml`

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
        android:strokeColor="@color/text_heading"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M19,12H5 M12,5L5,12L12,19" />
</vector>
```

#### `app/src/main/res/drawable/ic_list_grid.xml`

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
        android:strokeColor="@color/text_heading"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,3H9V9H3Z M15,3H21V9H15Z M3,15H9V21H3Z M15,15H21V21H15Z" />
</vector>
```

#### `app/src/main/res/drawable/ic_list_plus.xml`

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
        android:strokeColor="@color/text_heading"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,4V20 M4,12H20" />
</vector>
```

#### `app/src/main/res/drawable/ic_list_search.xml`

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
        android:strokeColor="@color/text_heading"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M10.5,3.5a7,7 0,1 0,0,14a7,7 0,1 0,0,-14 M16,16L21,21" />
</vector>
```

#### `app/src/main/res/drawable/ic_notification_close.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="2.4"
        android:strokeLineCap="round"
        android:pathData="M5,5L19,19 M19,5L5,19" />
</vector>
```

#### `app/src/main/res/drawable/ic_report_download.xml`

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
        android:pathData="M12,3V15 M7,10l5,5 5,-5 M4,17v3h16v-3"
        android:strokeColor="@color/white"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:strokeWidth="2" />
</vector>
```

#### `app/src/main/res/drawable/ic_user_calendar.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M5,5 H19 Q21,5 21,7 V20 Q21,22 19,22 H5 Q3,22 3,20 V7 Q3,5 5,5 Z M3,10 H21 M8,2 V7 M16,2 V7" />
</vector>
```

#### `app/src/main/res/drawable/ic_user_chevron.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M9,5 L16,12 L9,19" />
</vector>
```

#### `app/src/main/res/drawable/ic_user_email.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,5 H21 V19 H3 Z M3,5 L12,12 L21,5" />
</vector>
```

#### `app/src/main/res/drawable/ic_user_id.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M3,4 H21 V20 H3 Z M8,8 A2,2 0,1 1,8 12 A2,2 0,1 1,8 8 M5,17 C5,13 11,13 11,17 Z M14,8 H18 M14,12 H18 M14,16 H18" />
</vector>
```

#### `app/src/main/res/drawable/ic_user_location.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeWidth="1.8"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:pathData="M12,22 C9,18 4,13 4,9 A8,8 0,1 1,20 9 C20,13 15,18 12,22 Z M12,6 A3,3 0,1 1,12 12 A3,3 0,1 1,12 6" />
</vector>
```

#### `app/src/main/res/drawable/item_action_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/white_25">
    <item>
        <shape>
            <corners
                android:radius="100dp" />
            <gradient
                android:angle="315"
                android:startColor="@color/primary"
                android:endColor="@color/success" />
        </shape>
    </item>
</ripple>
```

#### `app/src/main/res/drawable/item_info_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <solid
        android:color="@color/primary_light" />
    <corners
        android:radius="100dp" />
</shape>
```

#### `app/src/main/res/drawable/menu_search_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="100dp" />
    <solid
        android:color="@color/search_background" />
</shape>
```

#### `app/src/main/res/drawable/modal_cancel.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/success_9">
    <item>
        <shape>
            <corners
                android:radius="12dp" />
            <solid
                android:color="@color/white_38" />
            <stroke
                android:width="1dp"
                android:color="@color/primary_light" />
        </shape>
    </item>
</ripple>
```

#### `app/src/main/res/drawable/modal_card.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <corners
        android:radius="24dp" />
    <gradient
        android:angle="225"
        android:startColor="@color/primary_light"
        android:centerColor="@color/primary_light"
        android:endColor="@color/white" />
</shape>
```

#### `app/src/main/res/drawable/modal_confirm.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/white_25">
    <item>
        <shape>
            <corners
                android:radius="12dp" />
            <gradient
                android:angle="270"
                android:startColor="@color/primary"
                android:endColor="@color/success" />
        </shape>
    </item>
</ripple>
```

#### `app/src/main/res/drawable/modal_danger.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/white_25">
    <item>
        <shape>
            <corners
                android:radius="12dp" />
            <gradient
                android:angle="270"
                android:startColor="@color/destructive"
                android:endColor="@color/badge_red" />
        </shape>
    </item>
</ripple>
```

#### `app/src/main/res/drawable/person_facts_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/field_background" />
    <corners android:radius="14dp" />
</shape>
```

#### `app/src/main/res/drawable/person_tag_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/primary_light" />
    <corners android:radius="100dp" />
</shape>
```

#### `app/src/main/res/drawable/person_tag_secondary_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/field_background" />
    <corners android:radius="100dp" />
</shape>
```

#### `app/src/main/res/drawable/red_badge.xml`

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
                android:startColor="@color/white"
                android:endColor="@color/white" />
            <stroke
                android:width="1dp"
                android:color="@color/destructive" />
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
                android:color="@color/destructive" />
        </shape>
    </item>
</layer-list>
```

#### `app/src/main/res/drawable/report_download_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/white_20">
    <item>
        <shape>
            <corners
                android:radius="16dp" />
            <gradient
                android:angle="90"
                android:startColor="@color/primary"
                android:endColor="@color/success" />
        </shape>
    </item>
</ripple>
```

#### `app/src/main/res/drawable/status_danger_circle.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape
            android:shape="oval">
            <solid
                android:color="@color/white" />
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
                android:color="@color/badge_red" />
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
                android:startColor="@color/destructive"
                android:endColor="@color/badge_red" />
        </shape>
    </item>
</layer-list>
```

#### `app/src/main/res/drawable/status_success_circle.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape
            android:shape="oval">
            <solid
                android:color="@color/primary_light" />
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
                android:color="@color/primary_light" />
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
                android:startColor="@color/primary"
                android:endColor="@color/success" />
        </shape>
    </item>
</layer-list>
```

#### `app/src/main/res/drawable/user_avatar_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <solid
        android:color="@color/primary_darkest" />
    <corners
        android:radius="100dp" />
</shape>
```

#### `app/src/main/res/drawable/user_detail_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android">
    <solid
        android:color="@color/primary_light" />
    <corners
        android:radius="100dp" />
</shape>
```

#### `app/src/main/res/drawable/auth_back_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/white_20" />
    <corners android:radius="12dp" />
</shape>
```

#### `app/src/main/res/drawable/auth_button_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/primary_darkest" />
    <corners android:radius="28dp" />
</shape>
```

#### `app/src/main/res/drawable/auth_field_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/field_background" />
    <corners android:radius="26dp" />
    <stroke
        android:width="1dp"
        android:color="@color/field_border" />
</shape>
```

#### `app/src/main/res/drawable/auth_hero_gradient.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <gradient
        android:angle="0"
        android:startColor="@color/primary_darkest"
        android:centerColor="@color/primary_dark"
        android:endColor="@color/primary" />
</shape>
```

#### `app/src/main/res/drawable/auth_sheet_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/white" />
    <corners
        android:topLeftRadius="34dp"
        android:topRightRadius="34dp" />
</shape>
```

#### `app/src/main/res/drawable/ic_auth_arrow.xml`

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
        android:strokeColor="@color/white"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:strokeWidth="2"
        android:pathData="M4,12h15 M13,6l6,6 -6,6" />
</vector>
```

#### `app/src/main/res/drawable/ic_auth_eye.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:strokeWidth="1.8"
        android:pathData="M2,12C4.5,7.5 7.8,5.5 12,5.5S19.5,7.5 22,12C19.5,16.5 16.2,18.5 12,18.5S4.5,16.5 2,12z M12,9a3,3 0,1 0,0 6a3,3 0,0 0,0 -6z" />
</vector>
```

#### `app/src/main/res/drawable/ic_auth_lock.xml`

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
        android:strokeColor="@color/text_body"
        android:strokeLineCap="round"
        android:strokeLineJoin="round"
        android:strokeWidth="1.8"
        android:pathData="M5,10h14v11H5z M8,10V7a4,4 0,0 1,8 0v3 M12,14v3" />
</vector>
```

#### `app/src/main/res/drawable/admin_account_menu_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/white" />
    <corners android:radius="16dp" />
    <stroke
        android:width="1dp"
        android:color="@color/search_border" />
</shape>
```

### res/color (4 files)

#### `app/src/main/res/color/admin_status_chip_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_checked="true"
        android:color="@color/primary_darkest" />
    <item
        android:color="@color/primary_light" />
</selector>
```

#### `app/src/main/res/color/admin_status_chip_text.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_checked="true"
        android:color="@color/white" />
    <item
        android:color="@color/text_body" />
</selector>
```

#### `app/src/main/res/color/dashboard_bell_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_selected="true"
        android:color="@color/primary_light" />
    <item
        android:color="@color/primary_light" />
</selector>
```

#### `app/src/main/res/color/dashboard_bell_tint.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:state_selected="true"
        android:color="@color/success" />
    <item
        android:color="@color/text_heading" />
</selector>
```

#### `app/src/main/res/drawable/item_show_icon_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="@color/white" />
</shape>
```

#### `app/src/main/res/drawable/item_show_sheet_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="@color/white" />
    <corners
        android:topLeftRadius="30dp"
        android:topRightRadius="30dp" />
</shape>
```

#### `app/src/main/res/drawable/profile_cover_background.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <gradient
        android:angle="0"
        android:startColor="@color/primary_darkest"
        android:centerColor="@color/primary_dark"
        android:endColor="@color/primary" />
</shape>
```

### res/values (27 files)

#### `app/src/main/res/values/personal_dashboard_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="personal_dashboard_title">My Dashboard</string>
    <string name="personal_dashboard_subtitle">Explore what matters to you.</string>
    <string name="personal_dashboard_search">Search anything...</string>
    <string name="personal_dashboard_notifications">Notifications</string>
    <string name="personal_dashboard_account">Account</string>
    <string name="personal_dashboard_initials">AR</string>
    <string name="personal_profile_title">My Profile</string>
    <string name="personal_profile_name">Alex Rivera</string>
    <string name="personal_profile_role">Member</string>
    <string name="personal_profile_access">Personal dashboard</string>
    <string name="personal_profile_status">Active</string>
    <string name="personal_notification_reservation_title">Reservation confirmed</string>
    <string name="personal_notification_reservation_message">Your table is ready for your visit.</string>
    <string name="personal_notification_item_title">New item to explore</string>
    <string name="personal_notification_item_message">Take a look at what is new on the menu.</string>
    <string name="personal_notification_appointment_title">Appointment reminder</string>
    <string name="personal_notification_appointment_message">You have an upcoming appointment.</string>
    <string name="personal_notification_now">Just now</string>
    <string name="personal_notification_hour">1 hour ago</string>
    <string name="personal_notification_yesterday">Yesterday</string>
    <string name="personal_featured_title">Featured Items</string>
    <string name="personal_featured_subtitle">Picked for you to explore</string>
    <string name="personal_activities_title">Activities</string>
    <string name="personal_activities_subtitle">Recent invitations and activities</string>
    <string name="personal_activities_search_hint">Search activities…</string>
    <string name="personal_activity_filter_all">All</string>
    <string name="personal_activity_filter_online">Online</string>
    <string name="personal_activity_filter_in_person">In person</string>
    <string name="personal_search_title">Explore items</string>
    <string name="personal_search_hint">Search items…</string>
    <string name="personal_search_close_description">Close search</string>
    <string name="personal_search_all">All</string>
    <string name="personal_seat_three">3 Seaters</string>
    <string name="personal_seat_five">5 Seaters</string>
    <string name="personal_seat_six">6 Seaters</string>
    <string name="personal_seat_eight">8 Seaters</string>
</resources>
```

#### `app/src/main/res/values/admin_account_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="admin_account_menu">Open account menu</string>
    <string name="admin_profile_action">Profile</string>
    <string name="admin_logout_action">Logout</string>
    <string name="admin_profile_title">My Profile</string>
    <string name="admin_profile_name">Kevin Merico</string>
    <string name="admin_profile_role">Administrator</string>
    <string name="admin_profile_access">Admin dashboard</string>
    <string name="admin_profile_status">Active</string>
</resources>
```

#### `app/src/main/res/values/auth_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="auth_login_title">Welcome\nBack</string>
    <string name="auth_login_subtitle">Glad to see you again.\nSign in to continue your journey.</string>
    <string name="auth_signup_title">Create Your\nAccount</string>
    <string name="auth_signup_subtitle">Join us and start your journey today.</string>
    <string name="auth_first_name">First Name</string>
    <string name="auth_last_name">Last Name</string>
    <string name="auth_email">Email</string>
    <string name="auth_password">Password</string>
    <string name="auth_confirm_password">Confirm Password</string>
    <string name="auth_first_name_hint">Enter your first name</string>
    <string name="auth_last_name_hint">Enter your last name</string>
    <string name="auth_email_hint">Email address</string>
    <string name="auth_password_hint">Enter your password</string>
    <string name="auth_confirm_hint">Confirm your password</string>
    <string name="auth_forgot">Forgot Password?</string>
    <string name="auth_login_action">Sign In</string>
    <string name="auth_register_action">Create Account</string>
    <string name="auth_upload_title">Add profile photo</string>
    <string name="auth_upload_subtitle">Tap to choose an image</string>
    <string name="auth_back">Back</string>
    <string name="auth_sample_title">Password reset</string>
    <string name="auth_sample_message">Password reset is not available yet.</string>
    <string name="auth_required">Please fill in this field</string>
    <string name="auth_password_mismatch">Passwords do not match</string>
    <string name="auth_no_account">Don’t have an account?</string>
    <string name="auth_have_account">Already have an account?</string>
    <string name="auth_signup_link">Sign Up</string>
    <string name="auth_signin_link">Sign In</string>
    <string name="auth_profile_photo_label">Profile Photo</string>
    <string name="auth_login_failed">Sign in failed</string>
    <string name="auth_login_failed_message">Check your email and password.</string>
    <string name="auth_email_exists">This email is already registered</string>
    <string name="auth_demo_credentials">Demo: morgan.lee@example.com (Admin) or alex.rivera@example.com (Personal). Password: 1234</string>
</resources>
```

#### `app/src/main/res/values/activity_card_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string
        name="sample_activity_title">You’re invited to Web Development Class</string>
    <string
        name="sample_activity_subtitle">by Alex Rivera</string>
    <string
        name="sample_activity_type">Invitation</string>
    <string
        name="sample_activity_date">Feb 5, 2024</string>
    <string
        name="sample_activity_time">6:00 PM – 7:00 PM</string>
    <string
        name="sample_activity_mode">Online</string>
    <string
        name="sample_activity_initials">AR</string>
    <string
        name="sample_activity_organizer">Alex Rivera</string>
    <string
        name="sample_activity_role">Organizer</string>
    <string
        name="sample_activity_secondary">Decline</string>
    <string
        name="sample_activity_primary">Accept</string>
    <string name="activity_accept_title">Accept invitation?</string>
    <string name="activity_decline_title">Decline invitation?</string>
    <string name="activity_accepted">Invitation accepted</string>
    <string name="activity_declined">Invitation declined</string>
</resources>
```

#### `app/src/main/res/values/activity_show_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="activity_show_reference">#ACT-%1$05d</string>
    <string name="activity_show_sample_reference">#ACT-00001</string>
    <string name="activity_show_overview">Activity details</string>
    <string name="activity_show_schedule">Schedule details</string>
    <string name="activity_show_payment">Payment details</string>
    <string name="activity_show_notes">Comments</string>
    <string name="activity_show_title_label">Activity</string>
    <string name="activity_show_organizer_label">Organizer</string>
    <string name="activity_show_date_label">Date</string>
    <string name="activity_show_time_label">Time</string>
    <string name="activity_show_mode_label">Location</string>
    <string name="activity_show_payment_method">Payment method</string>
    <string name="activity_show_amount">Amount</string>
    <string name="activity_show_total">Total</string>
    <string name="activity_show_not_required">Not available</string>
    <string name="activity_show_dash">—</string>
    <string name="activity_show_no_notes">No additional notes for this activity.</string>
    <string name="activity_show_hero_title">Activity overview</string>
    <string name="activity_show_hero_subtitle">Details and schedule at a glance</string>
    <string name="activity_show_icon_description">Activity calendar</string>
</resources>
```

#### `app/src/main/res/values/activity_show_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="ActivityShowCard">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_marginTop">12dp</item>
        <item name="android:background">@drawable/admin_outline_background</item>
        <item name="android:orientation">vertical</item>
        <item name="android:padding">16dp</item>
    </style>
    <style name="ActivityShowHeading">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:fontFamily">@font/arial_rounded_mt_bold</item>
        <item name="android:textColor">@color/primary_dark</item>
        <item name="android:textSize">@dimen/type_body_large</item>
    </style>
    <style name="ActivityShowRow">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_marginTop">14dp</item>
        <item name="android:gravity">center_vertical</item>
        <item name="android:orientation">horizontal</item>
    </style>
    <style name="ActivityShowLabel">
        <item name="android:layout_width">0dp</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_weight">1</item>
        <item name="android:textColor">@color/text_subheading</item>
        <item name="android:textSize">@dimen/type_subheading</item>
    </style>
    <style name="ActivityShowValue">
        <item name="android:layout_width">wrap_content</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:textColor">@color/text_heading</item>
        <item name="android:textSize">@dimen/type_subheading</item>
    </style>
    <style name="ActivityShowValueBadge" parent="green_badge">
        <item name="android:layout_marginStart">8dp</item>
        <item name="android:maxWidth">220dp</item>
        <item name="android:ellipsize">end</item>
        <item name="android:singleLine">true</item>
    </style>
</resources>
```

#### `app/src/main/res/values/admin_dashboard_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string
        name="admin_hello">Hello</string>
    <string
        name="admin_name">Kevin Merico!</string>
    <string
        name="admin_initials">KM</string>
    <string
        name="admin_subtitle">Keep managing your sales with care.</string>
    <string
        name="admin_search_hint">Search transactions...</string>
    <string name="admin_items_metric_title">Items</string>
    <string name="admin_persons_metric_title">Persons</string>
    <string name="admin_items_metric_action">View item list  ›</string>
    <string name="admin_persons_metric_action">View person list  ›</string>
    <string
        name="admin_zero_items">0 items</string>
    <string name="admin_sample_items">Items</string>
    <string name="admin_sample_items_subheading">Sample items available to browse</string>
    <string name="admin_sample_activities">Activities</string>
    <string name="admin_sample_activities_subheading">Recent invitations and activities</string>
    <string name="admin_activities_search_hint">Search activities...</string>
    <string name="admin_activities_online">Online</string>
    <string name="admin_activities_in_person">In person</string>
    <string name="admin_items_count">%1$d items</string>
    <string
        name="empty_list_title">Oops, the list is empty</string>
    <string
        name="empty_list_message">Items will appear here when available.</string>
    <string
        name="admin_pending">Pending</string>
    <string
        name="admin_all">All</string>
    <string
        name="admin_active">Active</string>
    <string
        name="admin_inactive">Inactive</string>
    <string
        name="admin_rejected">Rejected</string>
    <string
        name="admin_cancelled">Cancelled</string>
</resources>
```

#### `app/src/main/res/values/admin_dashboard_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style
        name="AdminHeading">
        <item
            name="android:layout_width">match_parent</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:textColor">@color/text_heading</item>
        <item
            name="android:textSize">@dimen/type_dashboard_greeting</item>
        <item
            name="android:fontFamily">@font/arial_rounded_mt_bold</item>
    </style>
    <style
        name="AdminMetricCard">
        <item
            name="android:layout_width">0dp</item>
        <item
            name="android:layout_height">142dp</item>
        <item
            name="android:layout_weight">1</item>
        <item
            name="android:orientation">vertical</item>
        <item
            name="android:padding">14dp</item>
        <item
            name="android:background">@drawable/admin_metric_background</item>
    </style>
    <style
        name="AdminMetricFloatingCard"
        parent="AdminMetricCard">
        <item
            name="android:elevation">10dp</item>
    </style>
    <style
        name="AdminMetricLabel">
        <item
            name="android:layout_width">match_parent</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:textColor">@color/primary_darkest</item>
        <item
            name="android:textSize">@dimen/type_subheading</item>
        <item
            name="android:fontFamily">@font/arial_rounded_mt_bold</item>
    </style>
    <style
        name="AdminMetricValue">
        <item
            name="android:layout_width">match_parent</item>
        <item
            name="android:layout_height">0dp</item>
        <item
            name="android:layout_weight">1</item>
        <item
            name="android:layout_marginTop">18dp</item>
        <item
            name="android:textColor">@color/primary_darkest</item>
        <item
            name="android:textSize">@dimen/type_metric</item>
        <item
            name="android:fontFamily">@font/arial_rounded_mt_bold</item>
    </style>
    <style
        name="AdminMetricChange">
        <item
            name="android:layout_width">match_parent</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:textSize">@dimen/type_detail</item>
    </style>
    <style
        name="AdminStatusChip"
        parent="Widget.Material3.Chip.Filter">
        <item
            name="android:layout_width">wrap_content</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:textSize">@dimen/type_label</item>
        <item
            name="android:fontFamily">@font/arial_rounded_mt_bold</item>
        <item
            name="android:textColor">@color/admin_status_chip_text</item>
        <item
            name="chipBackgroundColor">@color/admin_status_chip_background</item>
        <item
            name="chipCornerRadius">100dp</item>
        <item
            name="chipStrokeWidth">0dp</item>
        <item
            name="chipMinHeight">36dp</item>
        <item
            name="chipIconVisible">false</item>
        <item
            name="checkedIconVisible">false</item>
        <item
            name="ensureMinTouchTargetSize">false</item>
    </style>
    <style
        name="AdminStatusChipTheme">
        <item
            name="chipStyle">@style/AdminStatusChip</item>
    </style>
</resources>
```

#### `app/src/main/res/values/badge_styles.xml`

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
            name="android:textSize">@dimen/type_label</item>
        <item
            name="android:textStyle">bold</item>
    </style>
    <style
        name="green_badge"
        parent="StatusBadge">
        <item
            name="android:background">@drawable/green_badge</item>
        <item
            name="android:textColor">@color/badge_green</item>
    </style>
</resources>
```

#### `app/src/main/res/values/colors.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="primary_darkest">#062F1D</color>
    <color name="primary_dark">#08783F</color>
    <color name="primary">#25CE99</color>
    <color name="primary_lime">#BEF264</color>
    <color name="primary_light">#E6F7EE</color>

    <color name="success">#16B981</color>
    <color name="destructive">#EF4260</color>

    <color name="badge_green">#12805C</color>
    <color name="badge_red">#BC3048</color>
    <color name="badge_yellow">#F9A93D</color>

    <color name="text_heading">#101512</color>
    <color name="text_body">#526572</color>
    <color name="text_subheading">#84918A</color>

    <color name="white">#FFFFFFFF</color>
    <color name="transparent">#00000000</color>
    <color name="white_20">#33FFFFFF</color>
    <color name="white_25">#40FFFFFF</color>
    <color name="white_38">#60FFFFFF</color>
    <color name="black_27">#44000000</color>
    <color name="success_9">#1816B981</color>

    <color name="field_background">#F8F8F8</color>
    <color name="field_border">#E9E9E9</color>
    <color name="field_text">#171A18</color>
    <color name="field_hint">#999D9A</color>
    <color name="field_icon">#858B87</color>
    <color name="field_error">#C83F55</color>
    <color name="field_upload_border">#C8CECB</color>
    <color name="search_background">#F5F6F8</color>
    <color name="search_border">#E8ECE9</color>
    <color name="search_text">#142119</color>
    <color name="search_hint">#A7ABA8</color>
    <color name="create_page_background">#F5F6F5</color>
    <color name="activity_show_background">#F7FAF8</color>
    <color name="page_divider">#ECEEEC</color>
</resources>
```

#### `app/src/main/res/values/create_form_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="create_page_title">Create Entry</string>
    <string name="create_page_heading">Create New Entry</string>
    <string name="create_page_subtitle">Fill in the details below.</string>
    <string name="create_image_hint">Tap to choose an image</string>
    <string name="create_image_note">JPG or PNG image</string>
    <string name="create_name_label">Name *</string>
    <string name="create_name_hint">Enter a name</string>
    <string name="create_number_label">Number</string>
    <string name="create_number_hint">Enter a number</string>
    <string name="create_description_label">Description</string>
    <string name="create_description_hint">Write a short description</string>
    <string name="create_tags_label">Tags (select multiple)</string>
    <string name="create_tag_design">Design</string>
    <string name="create_tag_marketing">Marketing</string>
    <string name="create_tag_documentation">Documentation</string>
    <string name="create_date_label">Date</string>
    <string name="create_date_hint">Pick a date</string>
    <string name="create_time_label">Time</string>
    <string name="create_time_hint">Pick a time</string>
    <string name="create_option_label">Category</string>
    <string name="create_radio_label">Status</string>
    <string name="create_radio_active">Active</string>
    <string name="create_radio_inactive">Inactive</string>
    <string name="create_checkbox">Make this visible</string>
    <string name="create_save">Save Entry</string>
    <string name="create_name_required">Name is required</string>
    <string name="create_confirm_title">Continue with this entry?</string>
    <string name="create_confirm_message">Review the details before continuing.</string>
    <string name="create_done_title">Form complete</string>
    <string name="create_done_message">The sample form is ready to connect to your data.</string>
    <string-array name="create_categories">
        <item>General</item>
        <item>Food</item>
        <item>Service</item>
    </string-array>
</resources>
```

#### `app/src/main/res/values/create_form_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="CreateLabel">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_marginTop">14dp</item>
        <item name="android:textColor">@color/field_text</item>
        <item name="android:textSize">@dimen/type_subheading</item>
    </style>
    <style name="CreateField">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">44dp</item>
        <item name="android:layout_marginTop">6dp</item>
        <item name="android:background">@drawable/form_field_background</item>
        <item name="android:paddingHorizontal">14dp</item>
        <item name="android:textColor">@color/field_text</item>
        <item name="android:textColorHint">@color/field_hint</item>
        <item name="android:textSize">@dimen/type_subheading</item>
    </style>
    <style name="CreateFieldError">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_marginTop">4dp</item>
        <item name="android:textColor">@color/field_error</item>
        <item name="android:textSize">@dimen/type_detail</item>
    </style>
</resources>
```

#### `app/src/main/res/values/dashboard_sample_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
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

#### `app/src/main/res/values/dashboard_sample_styles.xml`

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
            name="strokeColor">@color/primary_light</item>
        <item
            name="cardBackgroundColor">@color/primary_light</item>
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
            name="android:textSize">@dimen/type_display</item>
        <item
            name="android:textStyle">bold</item>
        <item
            name="android:textColor">@color/text_heading</item>
    </style>
    <style
        name="DashboardStatLabel"
        parent="">
        <item
            name="android:layout_width">match_parent</item>
        <item
            name="android:layout_height">wrap_content</item>
        <item
            name="android:textSize">@dimen/type_body</item>
        <item
            name="android:textColor">@color/text_subheading</item>
    </style>
</resources>
```

#### `app/src/main/res/values/item_show_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="item_show_sample_description">A fresh favorite made with quality ingredients.</string>
    <string name="item_show_detail_label">Details</string>
    <string name="item_show_category_label">Category</string>
    <string name="item_show_tag_label">Tag</string>
    <string name="item_show_status_label">Status</string>
    <string name="item_show_share">Share Item</string>
    <string name="item_show_close">Go Back</string>
    <string name="item_show_about">About this item</string>
    <string name="item_show_about_vehicle">About this vehicle</string>
    <string name="item_show_from">Starting from</string>
    <string name="item_show_favorite">Save item</string>
</resources>
```

#### `app/src/main/res/values/item_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string
        name="item_available">Available</string>
    <string
        name="item_unavailable">Unavailable</string>
    <string
        name="item_view_details">View Details</string>
    <string
        name="sample_item_name">Cheeseburger</string>
    <string
        name="sample_item_category">Fast Food</string>
    <string
        name="sample_item_detail">320 calories</string>
    <string
        name="sample_item_tag">Beef</string>
    <string
        name="sample_item_rating">4.8 (120)</string>
    <string
        name="sample_item_location">Manila, PH</string>
    <string
        name="sample_item_price">₱120</string>
    <string
        name="item_rating">%1$.1f (%2$d)</string>
    <string
        name="item_add_favorite">Add to favorites?</string>
    <string
        name="item_remove_favorite">Remove from favorites?</string>
    <string
        name="item_favorite_message">Update favorites for %1$s?</string>
    <string
        name="item_favorite_updated">Favorites updated</string>
    <string
        name="item_favorite_preview">This change is for the sample preview only.</string>
</resources>
```

#### `app/src/main/res/values/item_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style
        name="ItemPhotoShape"
        parent="">
        <item
            name="cornerSize">12dp</item>
    </style>

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
            name="android:gravity">center_vertical</item>
        <item
            name="android:textSize">@dimen/type_caption</item>
        <item
            name="android:includeFontPadding">false</item>
    </style>
    <style
        name="ItemChip.Info"
        parent="ItemChip">
        <item
            name="android:background">@drawable/item_info_background</item>
        <item
            name="android:textColor">@color/text_heading</item>
    </style>
    <style
        name="ItemChip.Tag"
        parent="ItemChip">
        <item
            name="android:background">@drawable/user_detail_background</item>
        <item
            name="android:backgroundTint">@color/primary_light</item>
        <item
            name="android:textColor">@color/primary_dark</item>
    </style>
</resources>
```

#### `app/src/main/res/values/menu_list_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string
        name="menu_title">Menu Items</string>
    <string
        name="menu_add">Add Item</string>
    <string
        name="menu_search">Search menu items…</string>
    <string
        name="menu_all">All  3</string>
    <string
        name="menu_food">Food  3</string>
    <string
        name="menu_drinks">Drinks  0</string>
    <string
        name="menu_desserts">Desserts  0</string>
    <string
        name="menu_breakfast">Breakfast  1</string>
    <string
        name="menu_lunch">Lunch  1</string>
    <string
        name="menu_dinner">Dinner  1</string>
    <string
        name="menu_preview">Sample screen</string>
</resources>
```

#### `app/src/main/res/values/modal_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style
        name="ThemeOverlay.RestaurantApp.Modal"
        parent="ThemeOverlay.Material3.MaterialAlertDialog">
        <item
            name="colorPrimary">@color/success</item>
        <item
            name="android:backgroundDimAmount">0.55</item>
    </style>
</resources>
```

#### `app/src/main/res/values/notification_actions.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="notification_remove_title">Remove notification?</string>
    <string name="notification_remove_message">This notification will be removed from the list.</string>
    <string name="notification_removed_title">Notification removed</string>
    <string name="notification_removed_message">The notification was removed.</string>
</resources>
```

#### `app/src/main/res/values/person_list_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="person_list_title">Persons</string>
    <string name="person_add">Add Person</string>
    <string name="person_search">Search people…</string>
    <string name="person_all">All</string>
    <string name="person_active">Active</string>
    <string name="person_inactive">Inactive</string>
    <string name="person_add_preview">The Add Person page can be created separately.</string>
    <string name="person_age_label">Age</string>
    <string name="person_gender_label">Sex</string>
    <string name="person_location_label">Location</string>
    <string name="person_credits_label">Credits</string>
    <string name="person_gender_sample">Female</string>
</resources>
```

#### `app/src/main/res/values/report_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="report_title">Report</string>
    <string name="report_total_count">Total Count</string>
    <string name="report_sample_total">565</string>
    <string name="report_breakdown">Breakdown</string>
    <string name="report_overview">Activity overview</string>
    <string name="report_overview_subtitle">A clear view of your latest results.</string>
    <string name="report_sample_label">Sample data</string>
    <string name="report_total_caption">Across all recorded statuses</string>
    <string name="report_breakdown_subtitle">See how each status contributes to the total.</string>
    <string name="report_note_title">About this report</string>
    <string name="report_note">Each bar shows its status count as a share of the total activity.</string>
    <string name="report_download_pdf">Download PDF</string>
</resources>
```

#### `app/src/main/res/values/strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string
        name="app_name">RestaurantApp</string>
    <string
        name="confirm_arrow">Confirm  →</string>
    <string
        name="sample_user_name">Alex Morgan</string>
    <string
        name="sample_user_role">Member</string>
    <string
        name="user_active">Active</string>
    <string
        name="user_inactive">Inactive</string>
    <string
        name="user_edit">Edit</string>
    <string
        name="user_delete">Delete</string>
    <string name="person_quick_actions">Quick Actions</string>
    <string
        name="user_delete_title">Delete user?</string>
    <string
        name="user_delete_message">Remove %1$s from this sample list?</string>
    <string
        name="user_edit_preview">Edit %1$s</string>
    <string
        name="user_edit_note">Connect this button to your edit screen.</string>
    <string
        name="sample_user_initials">AM</string>
    <string
        name="sample_user_credits">250 credits</string>
    <string
        name="sample_user_age">29 years old</string>
    <string
        name="sample_user_location">Manila, PH</string>
    <string
        name="sample_user_email">jamie.reyes@example.com</string>
    <string
        name="sample_user_joined">Joined Jan 12, 2024</string>
    <string
        name="sample_user_specialty">Certified Level 2</string>
    <string
        name="user_credits">%1$d credits</string>
    <string
        name="user_age">%1$d years old</string>
    <string
        name="user_joined">Joined %1$s</string>
</resources>
```

#### `app/src/main/res/values/themes.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources
    xmlns:tools="http://schemas.android.com/tools">
    <style
        name="Base.Theme.RestaurantApp"
        parent="Theme.Material3.DayNight.NoActionBar">
        <item name="android:fontFamily">@font/arial_rounded_mt_bold</item>
        <item name="fontFamily">@font/arial_rounded_mt_bold</item>
        <item name="colorPrimary">@color/primary_dark</item>
        <item name="colorOnPrimary">@color/white</item>
        <item name="colorSecondary">@color/primary</item>
        <item name="colorOnSecondary">@color/text_heading</item>
        <item name="colorSurface">@color/white</item>
        <item name="colorOnSurface">@color/text_heading</item>
        <item name="colorError">@color/destructive</item>
        <item name="colorOnError">@color/white</item>
        <item name="android:colorAccent">@color/primary</item>
    </style>
    <style
        name="Theme.RestaurantApp"
        parent="Base.Theme.RestaurantApp" />
</resources>
```

#### `app/src/main/res/values/typography.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <dimen name="type_micro">9sp</dimen>
    <dimen name="type_caption">10sp</dimen>
    <dimen name="type_detail">11sp</dimen>
    <dimen name="type_label">12sp</dimen>
    <dimen name="type_subheading">13sp</dimen>
    <dimen name="type_body">14sp</dimen>
    <dimen name="type_button">15sp</dimen>
    <dimen name="type_body_large">16sp</dimen>
    <dimen name="type_person_name">17sp</dimen>
    <dimen name="type_card_heading">18sp</dimen>
    <dimen name="type_card_heading_large">19sp</dimen>
    <dimen name="type_section_heading">20sp</dimen>
    <dimen name="type_report_section">21sp</dimen>
    <dimen name="type_page_heading">22sp</dimen>
    <dimen name="type_large_heading">24sp</dimen>
    <dimen name="type_report_title">25sp</dimen>
    <dimen name="type_metric">26sp</dimen>
    <dimen name="type_dashboard_greeting">27sp</dimen>
    <dimen name="type_display">28sp</dimen>
    <dimen name="type_modal_symbol">36sp</dimen>
    <dimen name="type_report_total">46sp</dimen>
    <dimen name="type_auth_hero">34sp</dimen>
</resources>
```

#### `app/src/main/res/values/user_styles.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
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
            name="android:gravity">center_vertical</item>
        <item
            name="android:textColor">@color/text_body</item>
        <item
            name="android:textSize">@dimen/type_detail</item>
        <item
            name="android:includeFontPadding">false</item>
    </style>
    <style
        name="UserAvatarShape"
        parent="">
        <item
            name="cornerSize">50%</item>
    </style>
    <style name="PersonFactColumn" parent="">
        <item name="android:layout_width">0dp</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_weight">1</item>
        <item name="android:orientation">vertical</item>
        <item name="android:paddingHorizontal">4dp</item>
    </style>
    <style name="PersonFactLabel" parent="">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:textColor">@color/text_subheading</item>
        <item name="android:textSize">@dimen/type_caption</item>
    </style>
    <style name="PersonFactValue" parent="">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:layout_marginTop">4dp</item>
        <item name="android:textColor">@color/text_heading</item>
        <item name="android:textSize">@dimen/type_detail</item>
    </style>
</resources>
```

#### `app/src/main/res/values/profile_strings.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="profile_public_heading">Public profile</string>
    <string name="profile_public_hint">Update the details shown on your profile.</string>
    <string name="profile_name_label">Full name *</string>
    <string name="profile_email_label">Email address *</string>
    <string name="profile_name_error">Enter your name</string>
    <string name="profile_email_error">Enter a valid email address</string>
    <string name="profile_preferences_heading">Preferences</string>
    <string name="profile_notifications_label">Notifications</string>
    <string name="profile_notifications_hint">Show account and activity updates.</string>
    <string name="profile_logout_label">Log out</string>
    <string name="profile_logout_hint">Return to the sign-in screen.</string>
    <string name="profile_cancel">Cancel</string>
    <string name="profile_save">Save changes</string>
    <string name="profile_saved_title">Profile saved</string>
    <string name="profile_saved_message">Your changes are ready.</string>
    <string name="profile_save_error_title">Save failed</string>
    <string name="profile_save_error_message">Please try again.</string>
    <string name="profile_save_title">Save profile changes?</string>
    <string name="profile_save_message">Your profile details will be updated on this device.</string>
    <string name="profile_role_label">Role</string>
    <string name="profile_access_label">Access</string>
    <string name="profile_status_label">Status</string>
    <string name="personal_profile_email">alex.rivera@example.com</string>
    <string name="admin_profile_email">kevin.merico@example.com</string>
</resources>
```

### res/values-night (1 files)

#### `app/src/main/res/values-night/themes.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources
    xmlns:tools="http://schemas.android.com/tools">
    <style
        name="Base.Theme.RestaurantApp"
        parent="Theme.Material3.DayNight.NoActionBar" />
</resources>
```

### res/mipmap-anydpi-v26 (2 files)

#### `app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon
    xmlns:android="http://schemas.android.com/apk/res/android">
    <background
        android:drawable="@drawable/ic_launcher_background" />
    <foreground
        android:drawable="@drawable/ic_launcher_foreground" />
    <monochrome
        android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
```

#### `app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon
    xmlns:android="http://schemas.android.com/apk/res/android">
    <background
        android:drawable="@drawable/ic_launcher_background" />
    <foreground
        android:drawable="@drawable/ic_launcher_foreground" />
    <monochrome
        android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>
```


### res/xml (2 files)

#### `app/src/main/res/xml/backup_rules.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content />
```

#### `app/src/main/res/xml/data_extraction_rules.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<data-extraction-rules>
    <cloud-backup />
</data-extraction-rules>
```

### AndroidManifest.xml (1 file)

#### `app/src/main/AndroidManifest.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.RestaurantApp">
        <activity
            android:name=".SampleLoginActivity"
            android:exported="true">
            <intent-filter>
                <action
                    android:name="android.intent.action.MAIN" />
                <category
                    android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity
            android:name=".SampleRegisterActivity"
            android:exported="false" />
        <activity
            android:name=".SampleActivityShowActivity"
            android:exported="false" />
        <activity
            android:name=".SampleReportPageActivity"
            android:exported="false" />
        <activity
            android:name=".SampleCreatePageActivity"
            android:exported="false" />
        <activity
            android:name=".DashboardSampleActivity"
            android:exported="false" />
        <activity
            android:name=".PersonalDashboardActivity"
            android:exported="false" />
        <activity
            android:name=".PersonalProfileActivity"
            android:exported="false" />
        <activity
            android:name=".admin.AdminMenuListActivity"
            android:exported="false" />
        <activity
            android:name=".admin.AdminItemShowActivity"
            android:exported="false" />
        <activity
            android:name=".admin.AdminPersonListActivity"
            android:exported="false" />
        <activity
            android:name=".admin.AdminDashboardActivity"
            android:exported="false" />
        <activity
            android:name=".admin.AdminProfileActivity"
            android:exported="false" />
    </application>
</manifest>
```
