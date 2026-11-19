# NutriPulse — Eat Smart. Live Healthy.

Android app in **Java + XML** (no Kotlin, no Compose) with a working login/registration
system, per-user wellness data, and Berry the virtual pet.

## Open & run
1. Android Studio (a 2026 release) → **File › Open** → select this `NutriPulse` folder.
2. Let Gradle sync (needs internet the first time: AGP 9.4.1, Gradle 9.6.0, JDK 17–25).
3. Run on an emulator or phone with **Android 8.0 (API 26)** or higher.

No API keys or `google-services.json` are needed: the app works **fully offline**.
Optional: add Firebase for cloud login, password-reset emails, backup and AI chat (see `FIREBASE_SETUP.md`).

## Authentication (offline: Room + PBKDF2)
| Feature | Where |
|---|---|
| Splash checks the session → Home or Login | `auth/SplashScreenActivity` |
| Login: email format, required fields, password toggle, Remember Me, generic error message, 5-attempt lockout (1 min) | `auth/LoginActivity`, `auth/AuthManager` |
| Register: all fields validated, duplicate email/username check, password policy + strength meter, confirm match, Terms checkbox | `auth/RegisterActivity` |
| On registration: user + Berry (Lv 1, 0 EXP) + daily goals + calorie/hydration goals + achievements + preferences, in one DB transaction | `data/UserInitializer` |
| Forgot password: verify email + username, set a new password (old one is never shown) | `auth/ForgotPasswordActivity` |
| Change password (needs current password) | Settings › App Settings |
| Logout with confirmation; clears session only, data kept | Settings › Log Out |
| Protected screens redirect to Login without a session | `ui/BaseAuthActivity` |

**Password storage:** PBKDF2-HMAC-SHA256, 120,000 iterations, random 16-byte salt per user,
constant-time comparison (`utils/PasswordUtils`). Plaintext passwords are never saved.
**Session:** only the user id is stored (`utils/SessionManager`). Remember Me = 30 days;
otherwise the session ends the next time the app is cold-started.
**Privacy:** every query in `data/WellnessRepository` uses the user id from the session,
never from screen input, so one account can never read another's records.
Backups of the database/session are disabled.

## Project structure
```
app/src/main/java/com/nutripulse/app/
  auth/      AuthManager, SplashScreenActivity, LoginActivity, RegisterActivity, ForgotPasswordActivity
  models/    User, Pet, MealRecord, MealPlan, WeightRecord, WaterRecord, StepRecord,
             DailyGoal, Achievement, DietaryPreference
  database/  AppDatabase, UserDao, PetDao, MealDao, TrackingDao, GoalDao, DayTotal
  utils/     SessionManager, ValidationUtils, PasswordUtils, DateUtils, AppExecutors, UiUtils
  data/      WellnessRepository, UserInitializer, FoodCatalog, FoodItem, PetStats, Achievements
  ui/        Home, MealLogger, MealPlanner, Progress, Pet, Settings, Insights, BottomNav, Dialogs...
  views/     LineChartView, ProgressRingView (custom drawn, no chart library)
```

## Screens
Home dashboard · Meal Logger (Meal/Snack/Drink, search, servings, macros) · Meal Planner
(week strip, editable plan, calorie ring) · Progress (weight/calories/steps/water charts,
7D–1Y) · My Pet (care activities, EXP, evolution at Lv 5 and 10) · Settings (profile photo,
health goals, dietary preferences) · Health Insights (8 achievements, evolution).

## Known limits
- Barcode lookup uses Open Food Facts (free community database): some local products may be missing.
- Without Firebase, Forgot Password works offline (email + username check); with Firebase it sends a real reset email.

## Firebase (optional, built in)
Drop `app/google-services.json` into the project and the app switches on cloud features
automatically. See `FIREBASE_SETUP.md` for the step-by-step setup.
