# Android Studio (Java + Firebase) — Medicine Reminder App
## Full Detailed Build Prompt

---

## OVERVIEW

Build a fully functional Android mobile application called **"Medicine Reminder"** using:
- **Language:** Java
- **IDE:** Android Studio (minSdk 26, targetSdk 34)
- **Backend/Database:** Firebase (Authentication + Firestore + Cloud Messaging)
- **Architecture:** MVVM (ViewModel + LiveData + Repository pattern)
- **UI:** XML layouts with Material Design 3 components

The app is a personal medicine tracker with guardian alerts, adherence reports, AI health chat, prescriptions storage, family profiles, SOS emergency, and notification reminders.

Do NOT include any phone frame, outer background gradient, or iPhone mockup. Build only the app content itself — what goes inside the screen.

---

## DESIGN SYSTEM (match exactly)

### Color Palette (define in `res/values/colors.xml`)
```xml
<!-- Primary -->
<color name="primary">#2563EB</color>
<color name="primary_light">#3B82F6</color>
<color name="secondary">#0EA5E9</color>
<color name="accent">#14B8A6</color>

<!-- Semantic -->
<color name="success">#22C55E</color>
<color name="warning">#F59E0B</color>
<color name="danger">#EF4444</color>

<!-- Background / Surface -->
<color name="bg">#F8FAFC</color>
<color name="card">#FFFFFF</color>
<color name="border">#E5E7EB</color>

<!-- Text -->
<color name="text_primary">#111827</color>
<color name="text_muted">#6B7280</color>

<!-- Dark Mode equivalents -->
<color name="bg_dark">#0D1117</color>
<color name="card_dark">#161B22</color>
<color name="border_dark">#30363D</color>
<color name="text_primary_dark">#E6EDF3</color>
<color name="text_muted_dark">#8B949E</color>

<!-- Status chips -->
<color name="status_taken_bg">#DCFCE7</color>
<color name="status_taken_text">#16A34A</color>
<color name="status_pending_bg">#FEF9C3</color>
<color name="status_pending_text">#B45309</color>
<color name="status_missed_bg">#FEE2E2</color>
<color name="status_missed_text">#DC2626</color>
<color name="status_skipped_bg">#F3F4F6</color>
<color name="status_skipped_text">#6B7280</color>

<!-- Medicine color dots -->
<color name="med_blue_bg">#DBEAFE</color>
<color name="med_green_bg">#DCFCE7</color>
<color name="med_yellow_bg">#FEF9C3</color>
<color name="med_red_bg">#FEE2E2</color>
<color name="med_purple_bg">#F3E8FF</color>
<color name="med_orange_bg">#FFEDD5</color>
<color name="med_sky_bg">#E0F2FE</color>
<color name="med_pink_bg">#FCE7F3</color>

<!-- Gradients (define as drawables) -->
<!-- Hero: #2563EB → #0EA5E9 (135deg) -->
<!-- Guardian hero: #7C3AED → #A855F7 -->
<!-- Warning/upcoming: #F59E0B → #F97316 -->
<!-- Auth background: #1E40AF → #2563EB → #0EA5E9 -->
```

### Typography (define in `res/values/type.xml` and use `styles.xml`)
```
Font family: Use system default (-apple-system equivalent = Roboto on Android, or import "Inter" from Google Fonts via downloadable font)

Heading large:   22sp, weight 800 (ExtraBold)
Heading medium:  20sp, weight 800
Heading small:   18sp, weight 700
Body large:      16sp, weight 400
Body medium:     15sp, weight 400 or 600
Body small:      13sp, weight 500
Label:           12sp, weight 700, ALL CAPS, letterSpacing 0.5
Tiny:            11sp, weight 500
Badge:           10-11sp, weight 700
```

### Shape / Corner Radius
```
card_radius:       18dp
button_radius:     14dp
chip_radius:       20dp (pill shape)
avatar_radius:     50% (circle)
medicine_dot:      13dp
input_radius:      12dp
```

### Elevation / Shadow
Cards: `elevation="4dp"`, `cardElevation="4dp"`
FABs: `elevation="8dp"`
Bottom Nav: top border 1dp `#E5E7EB`

### Spacing
Standard content padding: `16dp` horizontal
Card internal padding: `18dp`
Item gap: `12–14dp`
Section gap: `16dp`

---

## FIREBASE SETUP

### `google-services.json`
Instruct user to download from Firebase Console and place in `/app/` directory.

### Firebase Services Required
1. **Firebase Authentication** — Email/Password sign-in
2. **Cloud Firestore** — NoSQL document database
3. **Firebase Cloud Messaging (FCM)** — Push notifications
4. **Firebase Storage** — (optional) for prescription image uploads

### Firestore Data Structure
```
/users/{userId}/
    name: String
    email: String
    role: String ("patient" | "guardian")
    createdAt: Timestamp
    settings: Map {
        darkMode: Boolean,
        notifications: Boolean,
        guardianAlerts: Boolean
    }

/users/{userId}/medicines/{medicineId}/
    name: String
    dosage: String
    qty: Number
    time: String ("HH:mm" 24h format)
    startDate: String ("yyyy-MM-dd")
    endDate: String ("yyyy-MM-dd", optional)
    frequency: String ("daily" | "twice" | "weekly" | "asneeded")
    colorIndex: Number (0–7)
    notes: String
    takenToday: Boolean
    skippedToday: Boolean
    lastResetDate: String ("yyyy-MM-dd")
    createdAt: Timestamp

/users/{userId}/history/{historyId}/
    medName: String
    dosage: String
    status: String ("taken" | "missed" | "skipped")
    timestamp: Timestamp
    date: String ("yyyy-MM-dd")

/users/{userId}/guardians/{guardianId}/
    email: String
    name: String
    status: String ("active" | "pending")
    addedAt: Timestamp
    fcmToken: String (optional)

/users/{userId}/notifications/{notifId}/
    category: String ("reminder" | "guardian" | "refill" | "emergency")
    title: String
    message: String
    icon: String (emoji or resource name)
    read: Boolean
    timestamp: Timestamp

/users/{userId}/prescriptions/{rxId}/
    name: String
    imageUrl: String (Firebase Storage URL)
    addedAt: Timestamp

/users/{userId}/family/{familyId}/
    name: String
    relation: String
    emoji: String
    colorHex: String
    addedAt: Timestamp
```

### Firestore Security Rules
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

---

## APP STRUCTURE

### Package: `com.medireminder`

```
com.medireminder/
├── ui/
│   ├── auth/
│   │   ├── LoginActivity.java
│   │   └── RegisterActivity.java
│   ├── main/
│   │   └── MainActivity.java          ← hosts BottomNavigationView + NavHostFragment
│   ├── home/
│   │   └── HomeFragment.java
│   ├── medicines/
│   │   ├── MedicinesFragment.java
│   │   └── AddMedicineBottomSheet.java
│   ├── reports/
│   │   └── ReportsFragment.java
│   ├── guardian/
│   │   └── GuardianFragment.java
│   ├── settings/
│   │   └── SettingsFragment.java
│   ├── notifications/
│   │   └── NotificationsActivity.java
│   ├── prescriptions/
│   │   └── PrescriptionsActivity.java
│   ├── family/
│   │   └── FamilyActivity.java
│   ├── aichat/
│   │   └── AIChatActivity.java
│   └── sos/
│       └── SOSActivity.java
├── viewmodel/
│   ├── AuthViewModel.java
│   ├── MedicineViewModel.java
│   ├── ReportsViewModel.java
│   ├── GuardianViewModel.java
│   └── SettingsViewModel.java
├── repository/
│   ├── MedicineRepository.java
│   ├── HistoryRepository.java
│   ├── GuardianRepository.java
│   └── NotificationRepository.java
├── model/
│   ├── Medicine.java
│   ├── HistoryEntry.java
│   ├── Guardian.java
│   ├── AppNotification.java
│   ├── Prescription.java
│   ├── FamilyMember.java
│   └── User.java
├── adapter/
│   ├── MedicineAdapter.java
│   ├── TodayMedicineAdapter.java
│   ├── HistoryAdapter.java
│   ├── GuardianAdapter.java
│   ├── NotificationAdapter.java
│   ├── PrescriptionAdapter.java
│   ├── FamilyAdapter.java
│   └── ChatAdapter.java
├── service/
│   ├── ReminderService.java           ← AlarmManager-based reminder
│   ├── MidnightResetReceiver.java     ← BroadcastReceiver to reset daily status
│   └── FCMService.java                ← FirebaseMessagingService
├── util/
│   ├── DateUtils.java
│   ├── FirestoreUtils.java
│   ├── NotifUtils.java
│   └── AdherenceUtils.java
└── MedicineReminderApp.java           ← Application class
```

---

## GRADLE DEPENDENCIES (`app/build.gradle`)

```groovy
dependencies {
    // Firebase BOM (Bill of Materials)
    implementation platform('com.google.firebase:firebase-bom:32.7.0')
    implementation 'com.google.firebase:firebase-auth'
    implementation 'com.google.firebase:firebase-firestore'
    implementation 'com.google.firebase:firebase-messaging'
    implementation 'com.google.firebase:firebase-storage'

    // Material Design 3
    implementation 'com.google.android.material:material:1.11.0'

    // Navigation Component
    implementation 'androidx.navigation:navigation-fragment:2.7.6'
    implementation 'androidx.navigation:navigation-ui:2.7.6'

    // ViewModel + LiveData
    implementation 'androidx.lifecycle:lifecycle-viewmodel:2.7.0'
    implementation 'androidx.lifecycle:lifecycle-livedata:2.7.0'

    // RecyclerView
    implementation 'androidx.recyclerview:recyclerview:1.3.2'

    // CardView
    implementation 'androidx.cardview:cardview:1.0.0'

    // MPAndroidChart (for pie chart and bar chart in reports)
    implementation 'com.github.PhilJay:MPAndroidChart:v3.1.0'

    // Glide (for prescription images)
    implementation 'com.github.bumptech.glide:glide:4.16.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.16.0'

    // CircleImageView (for avatars)
    implementation 'de.hdodenhof:circleimageview:3.1.0'

    // WorkManager (for reliable background tasks)
    implementation 'androidx.work:work-runtime:2.9.0'

    // Lottie (for animations, e.g. SOS pulse)
    implementation 'com.airbnb.android:lottie:6.3.0'

    // Shimmer (for loading states)
    implementation 'com.facebook.shimmer:shimmer:0.5.0'

    // SwipeRefreshLayout
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0'
}
```

Also add in project-level `build.gradle`:
```groovy
maven { url 'https://jitpack.io' }  // for MPAndroidChart
```

---

## SCREEN-BY-SCREEN SPECIFICATIONS

---

### 1. SPLASH / LAUNCH
**File:** `SplashActivity.java` + `activity_splash.xml`

- Full-screen gradient background: `#1E40AF` → `#2563EB` → `#0EA5E9` (vertical, LinearGradient drawable)
- Centered logo icon: rounded square (22dp radius), semi-transparent white background, 💊 emoji or pill icon at 40sp
- App name: "Medicine Reminder" — 26sp, weight 800, white
- Tagline: "Smart Medicine Care With Guardian Protection" — 13sp, white 75% opacity
- On launch: check `FirebaseAuth.getCurrentUser()`. If logged in → go to `MainActivity`. Else → go to `LoginActivity`. Delay 1200ms with Handler.

---

### 2. LOGIN SCREEN
**File:** `LoginActivity.java` + `activity_login.xml`

**Background:** Full-screen gradient drawable `bg_auth_gradient.xml`:
```xml
<gradient android:startColor="#1E40AF" android:centerColor="#2563EB"
          android:endColor="#0EA5E9" android:angle="160" android:type="linear"/>
```

**Layout (ScrollView → LinearLayout, vertical, padding 40dp 28dp):**

**Logo section (center-aligned):**
- Rounded square ImageView (72×72dp, 22dp radius, white 20% alpha background, 1dp white 30% border): contains pill icon 32sp
- TextView: "Medicine Reminder" — 26sp, 700, white
- TextView: "Smart Medicine Care With Guardian Protection" — 13sp, white 75%

**Auth Card (MaterialCardView, white 12% alpha, blur not available on Android so use semi-transparent, 24dp radius, 1dp white 20% border, padding 28dp 24dp):**

- TextView: "Sign In" — 20sp, 700, white
- Error banner (initially GONE): red 25% alpha background, 10dp radius, red text, 13sp
- Success banner (initially GONE): green 25% alpha background

**Email field:**
- Label: "EMAIL ADDRESS" — 12sp, 700, white 80%, ALL CAPS, letterSpacing 0.3
- `TextInputLayout` + `TextInputEditText`:
  - Background: white 15% alpha, 14dp radius, 1dp white 25% border
  - Text: white, 15sp
  - Hint text color: white 50%
  - Input type: `textEmailAddress`
  - On focus: border becomes white 60%

**Password field:** same style, `textPassword` input type

**Sign In button:**
- Full width, 15dp padding, white background, primary blue text (#2563EB), 14dp radius, 16sp, 700
- On click: validate → `FirebaseAuth.signInWithEmailAndPassword()` → on success go to `MainActivity`
- Show progress indicator (ProgressBar) while loading

**Create Account button (ghost style):**
- Full width, transparent background, 1.5dp white 35% border, 14dp radius, white 85% text, 15sp
- On click: go to `RegisterActivity`

**Demo Account button:**
- Same ghost style but 13sp, 70% opacity
- On click: sign in with `demo@medireminder.com` / `demo123` (auto-create if not exists, seed sample data in Firestore)

---

### 3. REGISTER SCREEN
**File:** `RegisterActivity.java` + `activity_register.xml`

Same background and card style as login.

Fields (same input style as login):
1. **Full Name** — `textPersonName`
2. **Email Address** — `textEmailAddress`
3. **Password** — `textPassword`, min 6 chars validation
4. **Role** — Spinner/DropdownMenu: "Patient" | "Guardian / Caregiver"

**Validation:**
- All fields required
- Email format check
- Password >= 6 chars
- Check Firebase for duplicate email (show error from `FirebaseAuthUserCollisionException`)

**Buttons:**
- "Create Account" primary button → `FirebaseAuth.createUserWithEmailAndPassword()` → on success write user doc to Firestore → navigate to `MainActivity`
- "Already have an account? Sign In" ghost button → back to `LoginActivity`

---

### 4. MAIN ACTIVITY
**File:** `MainActivity.java` + `activity_main.xml`

**Layout:**
```xml
<ConstraintLayout>
  <NavHostFragment />           ← fills entire screen
  <BottomNavigationView />      ← anchored to bottom
</BottomNavigationView>
```

**BottomNavigationView:**
- Background: white 92% (MaterialShapeDrawable with elevation overlay disabled)
- Top border: 1dp `#E5E7EB`
- Height: 80dp (use `app:itemPaddingTop` and `app:itemPaddingBottom` to control spacing)
- Icons (use Vector drawables for each):
  1. 🏠 Home (`ic_home.xml`)
  2. 💊 Medicines (`ic_pill.xml`)
  3. 📊 Reports (`ic_chart_bar.xml`)
  4. 🛡️ Guardian (`ic_shield.xml`)
  5. ⚙️ Settings (`ic_settings.xml`)
- Label style: 10sp, weight 500, muted color normally, primary blue + weight 700 when active
- Active item: show a small 4dp dot indicator below the label (custom BottomNavigationView subclass or use `app:itemIndicatorColor`)
- Icon scale animation: active icon scales to 1.12x using `ObjectAnimator`

**Navigation Graph** (`res/navigation/nav_graph.xml`):
Fragments: `HomeFragment`, `MedicinesFragment`, `ReportsFragment`, `GuardianFragment`, `SettingsFragment`

**FAB (FloatingActionButton):**
- Shown only on Home and Medicines fragments
- Gradient background: custom drawable `#2563EB` → `#0EA5E9`
- White `+` icon, 52dp size
- `elevation="12dp"`
- Positioned at `bottom=90dp`, `end=20dp` from parent
- On click: opens `AddMedicineBottomSheet`
- Scale animation on click: `scaleX/scaleY` 1.0 → 0.92 → 1.0

**SOS FAB:**
- Shown only on Home fragment
- Red (`#EF4444`) circular button, 52dp
- "SOS" text or emergency icon, white, 12sp bold
- `bottom=90dp`, `start=20dp`
- Pulsing animation: `ObjectAnimator` on `alpha` or `scaleX/scaleY` cycling 1.0 → 1.15 → 1.0, repeat infinite, 2s duration
- On click: show `SOSBottomSheet` or full-screen SOS overlay

**Status Bar:** Set `WindowCompat.setDecorFitsSystemWindows(window, false)` for edge-to-edge; status bar color transparent, icons dark/light based on theme.

**Midnight Reset:** Schedule `AlarmManager` repeating alarm at 00:00:30 every day that triggers `MidnightResetReceiver` to reset all `takenToday = false`, `skippedToday = false` in Firestore.

---

### 5. HOME FRAGMENT
**File:** `HomeFragment.java` + `fragment_home.xml`

**Layout:** `NestedScrollView` (no padding bottom override needed, BottomNav overlaps — use `paddingBottom="80dp"`)

#### 5a. Header (not scrollable, pinned at top):
```
[Good Morning, {Name}! 👋]      [🔔 badge]  [Avatar circle]
[You have 3 medicines today]
```
- Greeting: 22sp, 700, text_primary. Dynamic: Morning < 12h, Afternoon < 17h, Evening >= 17h
- Subtitle: 13sp, text_muted
- Notification bell: 36dp circle card, shadow, relative badge (red circle, 14dp, white border, shows unread count)
- Avatar: 40dp circle, gradient background `#2563EB → #0EA5E9`, first letter of name, white 16sp 700

#### 5b. Hero Card:
`MaterialCardView`, 22dp radius, no elevation (use custom shadow or `cardElevation="0dp"` with shadow drawable), gradient background drawable `#2563EB → #0EA5E9`

Content (padding 22dp 20dp):
- "Today's Adherence" — 16sp, 600, white 85%
- Large percentage: 42sp, 800, white (e.g. "0%" → "75%")
- Date string: 12sp, white 75% (e.g. "Monday, June 13")
- Pill emoji decorative (positioned end, 52sp, white 18% alpha): use a `TextView` absolutely positioned at end

#### 5c. Upcoming Card (conditionally shown if pending meds exist):
`MaterialCardView`, gradient background `#F59E0B → #F97316`, 18dp radius

Layout (horizontal LinearLayout, padding 16dp 18dp, gap 14dp):
- Emoji icon: 32sp
- Info column:
  - "Next: {medicine name}" — 15sp, 700, white
  - "{dosage} · Scheduled at {time}" — 12sp, white 85%
- Time string at end: 14sp, 800, white

#### 5d. Stats Row:
2×2 `GridLayout` (or `RecyclerView` with GridLayoutManager 2 cols), gap 12dp

Each stat card (`MaterialCardView`, 16dp radius, white bg, shadow):
- Emoji: 24sp
- Value: 22sp, 800, text_primary
- Label: 11sp, text_muted, 500

Stats: Total Today / Taken / In Stock / Guardians

#### 5e. Adherence Score Card:
`MaterialCardView`, 18dp radius, white bg, shadow

- "ADHERENCE SCORE" label — 13sp, 700, text_muted, ALL CAPS, letterSpacing 0.5
- Horizontal layout: [Circle Ring] [Info text]
- **Circular Ring View:** Custom `View` subclass `AdherenceRingView.java`:
  - 80dp × 80dp
  - Background ring: gray stroke, 8dp width, `radius=35`
  - Filled ring: green `#22C55E`, 8dp stroke, `stroke-linecap=round`, animated `strokeDashOffset` from 220 → calculated offset using `ValueAnimator` over 1200ms with `DecelerateInterpolator`
  - Center text: percentage, 18sp, 800, text_primary
- Info text:
  - Label: "Excellent! 🌟" / "Keep Going! 💪" / "Let's Improve! 📈" — 16sp, 700
  - Description: 12sp, text_muted

#### 5f. Today's Medicines Card:
`MaterialCardView`, 18dp radius

- "TODAY'S MEDICINES" label
- `RecyclerView` (nested scroll disabled, WRAP_CONTENT):
  - Each item (horizontal layout, padding 13dp vertical, bottom divider 1dp border):
    - Emoji dot: 44×44dp, 13dp radius, colored background
    - Medicine name: 15sp, 600
    - Dosage + time: 12sp, text_muted
    - Status chip (`MaterialChip` or styled `TextView`):
      - Taken: green bg `#DCFCE7`, green text `#16A34A`, "✓ Taken"
      - Pending: yellow bg `#FEF9C3`, amber text, "⏱ Pending"
      - Missed: red bg, red text, "Missed"
      - Skipped: gray bg, gray text, "Skipped"
  - Empty state: icon 48sp, "No Medicines Today", "Tap + to add your first medicine" — 13sp text_muted

---

### 6. MEDICINES FRAGMENT
**File:** `MedicinesFragment.java` + `fragment_medicines.xml`

**Header:**
- "My Medicines 💊" — 24sp, 800
- "{count} medicines tracked" — 13sp, text_muted

**Refill Alert Banner (conditionally shown):**
For each medicine with `qty <= 5`:
- Amber background `#FFF7ED`, 1dp amber border, 14dp radius, padding 12dp 14dp
- ⚠️ icon + "Refill Required" (13sp, 700, `#C2410C`) + "{name} — only {qty} left" (12sp, `#EA580C`)

**Medicine Cards** (`RecyclerView`, `LinearLayoutManager`):
Each card (`MaterialCardView`, 18dp radius, white bg, 1dp border `#E5E7EB`, shadow):
- Horizontal layout, gap 14dp, padding 16dp

Left: **Emoji dot** (50×50dp, 14dp radius, colored bg per `colorIndex`)

Center: **Info column**
- Medicine name: 16sp, 700
- Dosage + frequency: 12sp, text_muted
- Time: 11sp, primary blue, 600

Right: **Actions column**
- Stock badge: `#F8FAFC` bg, 1dp border, pill shape, "📦 {qty}", 11sp, text_muted — red tinted if `qty <= 5`
- "✓ Take" button: green bg `#DCFCE7`, green text, pill shape, 11sp, 700 — shown only if not taken today; replace with "✓ Done" green text if taken
- "Skip" button: gray bg, gray text, pill
- "🗑 Delete" button: red bg `#FEE2E2`, red text, pill

**Item Swipe (Optional Enhancement):** `ItemTouchHelper` for swipe-to-delete with undo `Snackbar`

**Click on card:** Opens `AddMedicineBottomSheet` pre-filled with medicine data for editing

---

### 7. ADD MEDICINE BOTTOM SHEET
**File:** `AddMedicineBottomSheet.java` + `bottom_sheet_add_medicine.xml`

`BottomSheetDialogFragment` with:
- Corner radius top: 28dp
- Drag handle: 36×4dp, `#E5E7EB`, centered, 8dp margin top, 18dp margin bottom
- Max height: 88% of screen height (set `peekHeight` and override `onCreateDialog`)
- Background: `bg` color (`#F8FAFC`)

**Title:** "Add Medicine 💊" or "Edit Medicine 💊" — 20sp, 800

**Form fields (same label style: 11sp, 700, text_muted, ALL CAPS, 0.4 letterSpacing):**
Each `TextInputLayout` with:
- `boxBackgroundMode="outline"`
- `boxStrokeColor`: border by default, primary on focus
- `boxCornerRadius`: 12dp
- Background: card white
- Text: text_primary, 15sp

1. **Medicine Name** (`textPersonName`)
2. **Dosage** (e.g. "500mg")
3. **Quantity** (`number`)
4. **Reminder Time** — `TimePicker` dialog on click (or `TextInputEditText` that opens `MaterialTimePicker`)
5. **Start Date** — `MaterialDatePicker` on click
6. **End Date** — `MaterialDatePicker` on click (optional)
7. **Frequency** — `AutoCompleteTextView` dropdown: Daily / Twice Daily / Weekly / As Needed
8. **Medicine Color** — horizontal `RecyclerView` of 8 color swatches (32dp circles, `colorIndex` 0–7):
   - Colors: `#DBEAFE`, `#DCFCE7`, `#FEF9C3`, `#FEE2E2`, `#F3E8FF`, `#FFEDD5`, `#E0F2FE`, `#FCE7F3`
   - Selected state: 3dp border in matching dark color + `scaleX/Y` 1.15
9. **Notes** — `TextInputEditText` multiline, min 3 lines, no resize

**Buttons (horizontal LinearLayout, gap 10dp, margin top 20dp):**
- "Cancel": `CardView` button, white bg, 1.5dp border `#E5E7EB`, 13dp radius, 15sp, 600, text_muted
- "Save Medicine ✓": gradient drawable background `#2563EB → #0EA5E9`, 13dp radius, 16sp, 700, white

**Save action:**
1. Validate required fields (name, dosage, qty, time) — show `Snackbar` on error
2. Create `Medicine` object with new `UUID`
3. Save to Firestore: `users/{uid}/medicines/{medId}`
4. Schedule `AlarmManager` reminder for the medicine time
5. Dismiss sheet
6. Show `Toast`/`Snackbar`: "💊 {name} saved!"
7. Refresh medicine list and home fragment

---

### 8. REPORTS FRAGMENT
**File:** `ReportsFragment.java` + `fragment_reports.xml`

**Header:** "Reports 📊" + "Your medicine analytics"

**Filter Tabs (HorizontalScrollView → LinearLayout):**
4 chips: Today / Weekly / Monthly / All Time
- Inactive: white bg, 1.5dp border `#E5E7EB`, text_muted, 12sp, 600, pill shape
- Active: primary blue bg `#2563EB`, white text, pill shape
- On click: filter history data and re-render all sections

**Stats Grid (2×2 GridLayout):**
Each `MaterialCardView` (15dp radius, center-aligned, shadow):
- Icon: 22sp
- Value: 24sp, 800
- Label: 11sp, text_muted

Stats: Total / Taken / Missed / Adherence%

**Adherence Breakdown Chart (`MaterialCardView`):**
Use `MPAndroidChart` — `PieChart`:
- No legend (custom legend below)
- Hole radius: 55%
- Hole color: card bg
- Segments: Taken (green `#22C55E`) / Missed (red `#EF4444`) / Skipped (gray `#9CA3AF`)
- No description label
- Animation: `animateY(1200ms, Easing.EaseInOutQuad)`
- Center text: adherence %
- Custom legend: 3 color dots + labels below chart

**Weekly Activity Chart (`MaterialCardView`):**
Use `MPAndroidChart` — `BarChart`:
- 7 bars for last 7 days (Sun–Sat)
- Stacked: taken (green `#4ADE80`) on top, missed (red `#FCA5A5`) below
- No grid lines (or very light)
- X axis: day abbreviations (S M T W T F S), 10sp, text_muted
- No left/right axis labels
- Bar corner radius: 3dp
- Animation: `animateY(800ms)`
- Empty bars (no data) shown as tiny gray stub

**Medicine History (`MaterialCardView`):**
`RecyclerView` of log items:
Each item (horizontal, 11dp padding vertical, bottom 1dp divider):
- Status icon square (36×36dp, 10dp radius, colored bg): ✅ green / ❌ red / ⏭ gray
- Medicine name: 14sp, 600
- Dosage + formatted datetime: 11sp, text_muted
- Status badge (pill chip): matching color

---

### 9. GUARDIAN FRAGMENT
**File:** `GuardianFragment.java` + `fragment_guardian.xml`

**Hero Card (`MaterialCardView`, 20dp radius, gradient bg `#7C3AED → #A855F7`):**
- "Guardian Protection" — 18sp, 700, white
- Description — 12sp, white 80%
- Decorative 🛡️ emoji positioned at end, 48sp, white 20%

**Add Guardian Card (`MaterialCardView`, 18dp radius):**
- "ADD GUARDIAN" label
- Email `TextInputLayout` (style same as form fields above)
- "Send Invite 📨" button: gradient blue, full width, 14dp padding

**My Guardians Card:**
`RecyclerView` of guardian items:
Each item (horizontal layout, padding 14dp vertical, bottom divider):
- Avatar circle (44dp, gradient purple, first letter, white 18sp 700)
- Name: 15sp, 600
- Email: 12sp, text_muted
- Status chip: Active (green bg/text) or Pending (yellow bg/text)
- Remove button: ✕, red, icon only

Empty state: "No guardians added yet." centered, 13sp, text_muted

**Recent Alerts Card:**
`RecyclerView` of guardian alert items:
Each item (horizontal, 12dp padding vertical, bottom divider):
- Icon square (38×38dp, 11dp radius, purple bg `#F3E8FF`, 🛡️)
- Alert message: 13sp, 600
- Formatted time: 11sp, text_muted

---

### 10. SETTINGS FRAGMENT
**File:** `SettingsFragment.java` + `fragment_settings.xml`

**Profile Header (`MaterialCardView`, 20dp radius, gradient bg `#2563EB → #0EA5E9`, center-aligned):**
- Avatar circle (72dp, white 25% bg, first letter, 28sp, 800, white, 3dp white 40% border)
- Name: 20sp, 700, white
- Email: 13sp, white 80%

**Settings Sections** (each `MaterialCardView`, 16dp radius):

**Profile Section:**
Each row (horizontal layout, padding 14dp 16dp, bottom 1dp divider, clickable with ripple):
- Icon (20sp, 32dp fixed width center-aligned)
- Title (15sp, 600) + Description (11sp, text_muted)
- Arrow "›" at end

Rows:
- 👤 Edit Profile → dialog to update name
- 📄 Prescriptions → `PrescriptionsActivity`
- 👨‍👩‍👧 Family Profiles → `FamilyActivity`
- 🤖 AI Health Assistant → `AIChatActivity`

**Preferences Section:**
Toggle rows (instead of arrow, show `SwitchMaterial` or custom toggle):

Custom toggle: `MaterialSwitch` or `Switch` styled:
- Track: gray `#D1D5DB` when off, green `#22C55E` when on
- Thumb: white, 22dp, shadow
- Track width: 48dp, Track height: 28dp, 20dp radius

Rows:
- 🌙 Dark Mode — `DayNight` theme switching via `AppCompatDelegate.setDefaultNightMode()`
- 🔔 Notifications — enable/disable `AlarmManager` reminders
- 🛡️ Guardian Alerts — enable/disable Firestore notification writes

**Account Section:**
- 🗑️ Clear All Data → confirmation `AlertDialog` → delete all Firestore subcollections
- 🚪 Sign Out (red title text) → `AlertDialog` confirmation → `FirebaseAuth.signOut()` → go to `LoginActivity`

**Footer:** "Medicine Reminder v1.0 • Made with ❤️" — 11sp, text_muted, centered, 12dp padding

---

### 11. PRESCRIPTIONS ACTIVITY
**File:** `PrescriptionsActivity.java` + `activity_prescriptions.xml`

**Toolbar (custom):**
- Back arrow `‹` (22sp, primary blue, back press)
- "Prescriptions 📄" title — 20sp, 800

**Upload Card (dashed border):**
`MaterialCardView` with dashed border drawable (2dp dashed, `#E5E7EB`), 18dp radius, center-aligned, 32dp padding:
- 📁 icon — 40sp
- "Upload Prescription" — 16sp, 700
- "Tap to select image from your device" — 12sp, text_muted
- On click: `Intent(Intent.ACTION_PICK)` for image gallery
- On result: upload to Firebase Storage at `users/{uid}/prescriptions/{uuid}.jpg`, save URL to Firestore
- Show `ProgressBar` while uploading

**Prescriptions List (`RecyclerView`):**
Each prescription card (`MaterialCardView`, 16dp radius):
- `ImageView` (full width, max height 180dp, `scaleType="centerCrop"`) loaded with Glide
- Footer (horizontal layout, padding 12dp 14dp):
  - File name: 14sp, 600
  - Date added: 11sp, text_muted
  - Delete button: red bg `#FEE2E2`, red text, 8dp radius, 13sp

Empty state: 📄 icon + "No Prescriptions" + "Upload your first prescription above"

---

### 12. FAMILY PROFILES ACTIVITY
**File:** `FamilyActivity.java` + `activity_family.xml`

**Toolbar:** Back + "Family Profiles 👨‍👩‍👧"

**Family List (`RecyclerView`):**
Each card (`MaterialCardView`, 18dp radius, horizontal layout, gap 14dp):
- Avatar circle (52dp, colored bg per relation): emoji inside 22sp
- Name: 16sp, 700
- Relation: 12sp, text_muted
- Arrow "›" at end
- Remove ✕ button: red icon, right side
- On click: (future detail screen or just show toast)

**Add Member Button (full width, dashed border `MaterialCardView`):**
- "＋ Add Family Member" — 15sp, 600, text_muted
- On click: `AlertDialog` with fields: Name, Relation (Spinner: Self/Father/Mother/Sibling/Spouse/Child/Other)
- On confirm: save to Firestore `users/{uid}/family/{id}`, refresh list

---

### 13. AI CHAT ACTIVITY
**File:** `AIChatActivity.java` + `activity_ai_chat.xml`

**Toolbar:**
- Back button
- Bot avatar circle (32dp, gradient purple `#7C3AED → #A855F7`, 🤖 icon)
- "Health Assistant" — 15sp, 700
- "● Online" — 11sp, green `#22C55E`, 600
- Bottom border: 1dp `#E5E7EB`

**Chat Messages (`RecyclerView`, `LinearLayoutManager` reverse=false, scroll to bottom on new message):**
`ChatAdapter` with 2 view types: USER and BOT

Bot bubble:
- White bg card, 18dp radius (bottom-left corner 4dp), shadow
- Text: 14sp, text_primary, lineHeight 1.5
- Align: start (left)

User bubble:
- Primary blue bg `#2563EB`, 18dp radius (bottom-right corner 4dp)
- Text: 14sp, white
- Align: end (right)

Message animation: `slideInFromBottom` (`TranslateAnimation`) + `fadeIn` on each new message

**Quick Questions (initially visible, hidden after first message):**
`LinearLayout` vertical, padding 0 12dp 10dp, gap 6dp:

5 pill-shaped outlined buttons:
- "💊 What is medicine adherence?"
- "⚠️ What if I miss a dose?"
- "🔔 How do reminders work?"
- "📈 How to improve adherence?"
- "🛡️ What is a guardian?"

Style: white bg, 1.5dp blue border, primary blue text, 12sp, 600, 20dp radius

**Input Row (pinned at bottom, white bg, top border 1dp):**
- `TextInputEditText`: flex 1, gray bg, 1.5dp border, 22dp radius, 14sp, "Ask about your medicines..."
- Send button: 40dp circle, primary blue, white ➤ icon
- `onEditorActionListener` for keyboard send

**AI Responses (local lookup map, no API needed):**
```java
Map<String, String> aiResponses = new HashMap<>();
aiResponses.put("medicine adherence", "Medicine adherence means taking your medications exactly as prescribed...");
aiResponses.put("miss a dose", "If you miss a dose: take it as soon as you remember...");
aiResponses.put("reminders work", "Medicine Reminder sends you a notification at the time you set...");
aiResponses.put("improve adherence", "Tips to improve adherence: 1) Set alarms that match your routine...");
aiResponses.put("what is a guardian", "A Guardian is a trusted person who monitors your medicine schedule...");
```
Default response: "That's a great question! As your Health Assistant, I can help with questions about your medicines..."

Use `Handler.postDelayed()` for 600–700ms bot "typing" delay before showing response.

**Init message on open:** "Hello! I'm your Medicine Reminder Health Assistant 🤖💊\n\nI can answer questions about medicine adherence, dosing, reminders, and how to use the app. How can I help you today?"

---

### 14. NOTIFICATIONS ACTIVITY
**File:** `NotificationsActivity.java` + `activity_notifications.xml`

**Toolbar:** Back + "Notifications 🔔" + "Clear All" button (right, primary blue, 13sp, 700)

On open: mark all notifications as `read=true` in Firestore, clear badge count

**Notifications List (`RecyclerView`):**
Grouped by category (use `StickyHeader` library or just `TextView` separators):
- 🆘 Emergency (shown first, red bg `#FEE2E2` for icon)
- 💊 Reminders (blue bg `#DBEAFE`)
- 🛡️ Guardian Alerts (purple bg `#F3E8FF`)
- 📦 Refill Alerts (amber bg `#FFF7ED`)

Each item (`MaterialCardView`, 14dp radius, left accent border 3dp primary blue if unread):
- Category icon square (36×36dp, 10dp radius)
- Title: 13sp, 700
- Message: 12sp, text_muted
- Time: 10sp, text_muted

Empty state: 🔔 + "No Notifications" + "You're all caught up!"

---

### 15. SOS SCREEN / OVERLAY
**File:** Implemented as a full-screen `DialogFragment` or second `Activity`

**Full-screen red overlay** (`#EF4444` at 96% alpha or solid):
- Center content:
  - Pulsing circle (custom `View` with `ObjectAnimator` on `alpha` 1.0→0.0→1.0, radius expanding): 140×140dp, white 15% bg, 🆘 icon 56sp inside
  - "SOS Alert Sent!" — 28sp, 800, white
  - "Emergency alert has been sent to all your guardians. Help is on the way." — 15sp, white 85%, center-aligned, padding 0 32dp
  - "I'm Okay — Dismiss" button: white 20% bg, 2dp white 50% border, pill shape, 16sp, 700, white

**On trigger:**
1. Write emergency notification to Firestore for current user
2. Send guardian alert notification to all `active` guardians (update their FCM token)
3. Show 3-second countdown before "Alert Sent" confirmation

---

### 16. REMINDER NOTIFICATION POPUP

**In-App Reminder Popup:** Custom `MaterialCardView` that slides in from top when a medicine time is reached:
- Position: `top=70dp`, `start=12dp`, `end=12dp`, `elevation=24dp`
- Animate in: `TranslateAnimation` from `-200dp` to `0` (spring interpolation)
- Auto-dismiss after 30 seconds

Layout:
- Top row: emoji dot icon (48×48dp, 14dp radius) + medicine name (16sp, 700) + detail (12sp, text_muted)
- 3-button grid:
  - "✓ Taken" — green bg `#DCFCE7`, green text
  - "5 min" — yellow bg, amber text
  - "Skip" — gray bg, gray text

**Actions:**
- Taken: call `markTaken(medId)` → log history, decrement qty, send guardian alert, dismiss popup
- 5 min: `Handler.postDelayed(showReminderPopup, 5 * 60 * 1000)`, dismiss, show Snackbar "Snoozed 5 minutes"
- Skip: call `markSkipped(medId)` → log history, dismiss popup

**System Notification (also shown when app is backgrounded):**
Use `NotificationCompat.Builder`:
- Channel: "medicine_reminders", importance HIGH
- Icon: pill icon drawable
- Title: "💊 Medicine Reminder"
- Content: "Time to take {name} ({dosage})"
- Big text style
- Action buttons: "✓ Taken", "Skip" (use `PendingIntent` with `BroadcastReceiver`)
- Auto cancel: false
- Priority: HIGH

---

## REMINDER / ALARM SYSTEM

### `ReminderService.java`
Use `AlarmManager` (not `WorkManager`, as it fires even in Doze mode with `setAlarmClock()`):

```java
// Schedule for each medicine
AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
Intent intent = new Intent(context, ReminderReceiver.class);
intent.putExtra("med_id", medicine.getId());
intent.putExtra("med_name", medicine.getName());
intent.putExtra("dosage", medicine.getDosage());

PendingIntent pendingIntent = PendingIntent.getBroadcast(
    context, requestCode, intent,
    PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE
);

// Parse medicine.time ("HH:mm"), build Calendar for today at that time
Calendar cal = Calendar.getInstance();
cal.set(Calendar.HOUR_OF_DAY, hour);
cal.set(Calendar.MINUTE, minute);
cal.set(Calendar.SECOND, 0);

// Use setAlarmClock for highest priority
alarmManager.setAlarmClock(
    new AlarmManager.AlarmClockInfo(cal.getTimeInMillis(), pendingIntent),
    pendingIntent
);
```

### `ReminderReceiver.java` (BroadcastReceiver)
- On receive: check if medicine is not already taken/skipped for today
- If not: fire system notification AND broadcast to app if foregrounded (use `LocalBroadcastManager`)
- Re-schedule for next occurrence (daily/weekly/twice)

### `MidnightResetReceiver.java` (BroadcastReceiver)
- On receive: query all medicines for current user, update `takenToday=false`, `skippedToday=false`, `lastResetDate=today` in Firestore
- Re-schedule itself for next midnight

---

## FCM (Firebase Cloud Messaging)

### `FCMService.java` (extends `FirebaseMessagingService`)
```java
@Override
public void onNewToken(String token) {
    // Save token to Firestore: users/{uid}/fcmToken = token
}

@Override
public void onMessageReceived(RemoteMessage message) {
    // Build and show notification from message.getNotification() or message.getData()
}
```

### Guardian Alerts via FCM:
When a user takes/misses a medicine:
1. Read guardians from Firestore
2. For each guardian with `status="active"` and `fcmToken` set:
3. Call Firebase HTTP v1 API or Cloud Functions (recommended) to send FCM to that token

**Cloud Function (Node.js)** `functions/index.js`:
```javascript
exports.sendGuardianAlert = functions.firestore
    .document('users/{userId}/history/{historyId}')
    .onCreate(async (snap, context) => {
        const data = snap.data();
        const userId = context.params.userId;
        // Get guardians, send FCM messages to each
    });
```

---

## DARK MODE

Support `DayNight` theme:
1. `AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES/NO)` based on user setting
2. Define `res/values-night/colors.xml` with dark equivalents:
   - `bg` → `#0D1117`
   - `card` → `#161B22`
   - `text_primary` → `#E6EDF3`
   - `text_muted` → `#8B949E`
   - `border` → `#30363D`
3. Auth background stays the same deep blue gradient
4. Bottom nav: `#161B22` at 95% alpha
5. Modal bottom sheets: `#0D1117` background

---

## ANIMATIONS

All transitions using `ObjectAnimator` or `ViewPropertyAnimator`:

1. **Page transitions (fragment navigation):**
   ```java
   NavOptions navOptions = new NavOptions.Builder()
       .setEnterAnim(R.anim.slide_in_right)
       .setExitAnim(R.anim.slide_out_left)
       .setPopEnterAnim(R.anim.slide_in_left)
       .setPopExitAnim(R.anim.slide_out_right)
       .build();
   ```
   Define `res/anim/slide_in_right.xml`: `translateX` from `20%` to `0` + `alpha` 0→1, 280ms `DecelerateInterpolator`

2. **Card fade-up animation:** `AlphaAnimation` + `TranslateAnimation` composed, 400ms, 0→1 opacity, 16dp→0dp Y translation, applied to each `RecyclerView` item via `LayoutAnimation`

3. **SOS pulse:** `ScaleAnimation` and `AlphaAnimation` on the outer ring, cycling 1.0→1.2→1.0 for scale, alpha 1.0→0.0, infinite repeat, 1500ms

4. **FAB press:** `ViewPropertyAnimator.scaleX(0.92).scaleY(0.92).setDuration(100)` then restore

5. **Bottom sheet slide:** Built into `BottomSheetDialogFragment` — override animation speed via `BottomSheetBehavior`

6. **Reminder popup slide-in:** `TranslateAnimation(0, 0, -500, 0)`, 500ms, `OvershootInterpolator(1.5f)`

7. **Adherence ring:** `ValueAnimator.ofFloat(220f, targetOffset)`, 1200ms, `DecelerateInterpolator`, update ring's `strokeDashOffset` equivalent via custom canvas drawing

---

## AndroidManifest.xml

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"/>
<uses-permission android:name="android.permission.USE_EXACT_ALARM"/>
<uses-permission android:name="android.permission.VIBRATE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32"/>
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>

<application
    android:name=".MedicineReminderApp"
    android:theme="@style/Theme.MedicineReminder"
    android:enableOnBackInvokedCallback="true">

    <activity android:name=".ui.auth.LoginActivity"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
    </activity>
    <activity android:name=".ui.main.MainActivity"/>
    <activity android:name=".ui.notifications.NotificationsActivity"/>
    <activity android:name=".ui.prescriptions.PrescriptionsActivity"/>
    <activity android:name=".ui.family.FamilyActivity"/>
    <activity android:name=".ui.aichat.AIChatActivity"/>

    <service
        android:name=".service.FCMService"
        android:exported="false">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT"/>
        </intent-filter>
    </service>

    <receiver android:name=".service.ReminderReceiver"
        android:exported="false"/>
    <receiver android:name=".service.MidnightResetReceiver"
        android:exported="false"/>
    <receiver android:name=".service.BootReceiver"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
    </receiver>
</application>
```

---

## STYLES & THEMES (`res/values/styles.xml`)

```xml
<style name="Theme.MedicineReminder" parent="Theme.Material3.DayNight.NoActionBar">
    <item name="colorPrimary">@color/primary</item>
    <item name="colorPrimaryVariant">@color/primary_light</item>
    <item name="colorSecondary">@color/secondary</item>
    <item name="colorSurface">@color/card</item>
    <item name="android:colorBackground">@color/bg</item>
    <item name="colorOnPrimary">@android:color/white</item>
    <item name="android:statusBarColor">@android:color/transparent</item>
    <item name="android:windowLightStatusBar">true</item>
    <item name="android:navigationBarColor">@android:color/transparent</item>
    <item name="shapeAppearanceMediumComponent">@style/ShapeAppearance.Card</item>
</style>

<style name="ShapeAppearance.Card" parent="ShapeAppearance.Material3.MediumComponent">
    <item name="cornerSize">18dp</item>
</style>

<!-- Input fields -->
<style name="Widget.MedicineReminder.TextInputLayout" parent="Widget.Material3.TextInputLayout.OutlinedBox">
    <item name="boxStrokeColor">@color/border</item>
    <item name="boxStrokeWidthFocused">1.5dp</item>
    <item name="boxCornerRadiusTopStart">12dp</item>
    <item name="boxCornerRadiusTopEnd">12dp</item>
    <item name="boxCornerRadiusBottomStart">12dp</item>
    <item name="boxCornerRadiusBottomEnd">12dp</item>
    <item name="hintTextColor">@color/text_muted</item>
</style>

<!-- Auth input fields (white style) -->
<style name="Widget.MedicineReminder.TextInputLayout.Auth" parent="Widget.MedicineReminder.TextInputLayout">
    <item name="boxStrokeColor">#40FFFFFF</item>
    <item name="hintTextColor">#80FFFFFF</item>
    <item name="android:textColorHint">#80FFFFFF</item>
</style>

<!-- Primary button -->
<style name="Widget.MedicineReminder.Button.Primary">
    <item name="android:background">@drawable/bg_btn_primary</item>
    <item name="android:textColor">@color/primary</item>
    <item name="android:textSize">16sp</item>
    <item name="android:textStyle">bold</item>
    <item name="android:padding">15dp</item>
</style>

<!-- Ghost button -->
<style name="Widget.MedicineReminder.Button.Ghost">
    <item name="android:background">@drawable/bg_btn_ghost</item>
    <item name="android:textColor">#D9FFFFFF</item>
    <item name="android:textSize">15sp</item>
    <item name="android:padding">13dp</item>
</style>
```

---

## KEY DRAWABLES REQUIRED

**`res/drawable/`:**
- `bg_auth_gradient.xml` — vertical blue gradient for auth screens
- `bg_hero_gradient.xml` — `#2563EB → #0EA5E9` for hero card
- `bg_guardian_gradient.xml` — `#7C3AED → #A855F7`
- `bg_upcoming_gradient.xml` — `#F59E0B → #F97316`
- `bg_btn_primary.xml` — white fill, 14dp corners
- `bg_btn_gradient.xml` — `#2563EB → #0EA5E9`, 14dp corners
- `bg_btn_ghost.xml` — transparent, 1.5dp white border, 14dp corners
- `bg_btn_taken.xml` — `#DCFCE7`, 20dp corners
- `bg_btn_skip.xml` — `#F3F4F6`, 20dp corners
- `bg_btn_delete.xml` — `#FEE2E2`, 20dp corners
- `bg_input_auth.xml` — white 15% fill, 14dp corners, 1dp white 25% border
- `bg_upload_dashed.xml` — dashed 2dp `#E5E7EB` border, 18dp corners
- `bg_toggle_on.xml` — `#22C55E` fill, 20dp radius (track shape)
- `bg_toggle_off.xml` — `#D1D5DB` fill, 20dp radius
- `ic_pill.xml` — Vector pill icon
- `ic_home.xml`, `ic_chart_bar.xml`, `ic_shield.xml`, `ic_settings.xml` — Vector icons

---

## BUILD FLOW CHECKLIST

Build in this order:

1. [ ] Create Android project (Empty Activity, Java, minSdk 26)
2. [ ] Add `google-services.json` from Firebase Console
3. [ ] Add all dependencies to `build.gradle` (app + project level)
4. [ ] Define `colors.xml`, `strings.xml`, `styles.xml`, `themes.xml`
5. [ ] Create all drawable resources
6. [ ] Create all `model/` POJOs with Firestore `@DocumentId` and proper constructors
7. [ ] Build `LoginActivity` + `RegisterActivity` with Firebase Auth
8. [ ] Build `MainActivity` with `BottomNavigationView` + Navigation Component
9. [ ] Build `HomeFragment` with all sections
10. [ ] Build `MedicinesFragment` + `AddMedicineBottomSheet`
11. [ ] Build `ReportsFragment` with MPAndroidChart
12. [ ] Build `GuardianFragment` + Firestore guardian logic
13. [ ] Build `SettingsFragment` + dark mode toggle
14. [ ] Build sub-activities: Prescriptions, Family, AI Chat, Notifications
15. [ ] Build `AdherenceRingView` custom view
16. [ ] Build `AlarmManager` reminder system
17. [ ] Build `FCMService` for push notifications
18. [ ] Build `MidnightResetReceiver` + `BootReceiver`
19. [ ] Add all animations
20. [ ] Test edge cases: no internet, empty states, permission denials
21. [ ] Dark mode testing on all screens

---

## NOTES FOR AI CODE GENERATION

- Every `FirebaseFirestore` call must be made off the main thread or handled with `addOnSuccessListener` / `addOnFailureListener` callbacks
- All `RecyclerView` adapters must call `notifyDataSetChanged()` or use `DiffUtil` for performance
- Use `LifecycleOwner` for `LiveData` observers in Fragments (use `getViewLifecycleOwner()`, not `this`)
- For `BottomSheetDialogFragment`, override `onCreateView` AND `onViewCreated`, not `onCreate`
- Auth state: use `FirebaseAuth.getInstance().addAuthStateListener()` in `LoginActivity` to auto-navigate if already signed in
- Prescription images: after picking from gallery, compress with `Bitmap.createScaledBitmap()` before uploading to Firebase Storage to save bandwidth
- For `TimePicker` in the add medicine form, use `MaterialTimePicker` from Material3 library
- All `PendingIntent` must include `FLAG_IMMUTABLE` (required Android 12+)
- `AlarmManager.setExactAndAllowWhileIdle()` or `setAlarmClock()` required for Android 6+ Doze mode
- Request `SCHEDULE_EXACT_ALARM` permission at runtime on Android 12+ via `Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM` intent
- Request `POST_NOTIFICATIONS` permission at runtime on Android 13+ using `ActivityCompat.requestPermissions()`       Adding Supabase As Firebase As Backend and Database
