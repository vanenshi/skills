# Persian (Farsi) UX Writing & Localization Rules

When generating, translating, or editing UX text for Persian, you must strictly follow these typographical, cultural, and structural rules. Do not simply translate English source strings word-for-word; localize them for the Persian user's mental model.

## 1. Typography and Mechanics (نگارش و تایپوگرافی)

Flawless typography builds trust. Persian UX writing requires strict adherence to specific character sets and spacing rules.

- **Zero-Width Non-Joiner (نیم‌فاصله - ZWNJ):** You MUST use ZWNJ correctly. Omitting it is a critical error in Persian UI.
  - **Plural suffixes (ها):** `آگهی‌ها` (Correct) | `آگهی ها` (Incorrect).
  - **Verbal prefixes (می / نمی‌):** `می‌روم` (Correct) | `می روم` (Incorrect).
  - **Compound words:** `جست‌وجو` (Correct) | `جست و جو` (Incorrect), `ثبت‌نام` (Correct) | `ثبت نام` (Incorrect).
  - **Adjective suffixes (تر / ترین):** `جدیدترین` (Correct) | `جدید ترین` (Incorrect).
- **Persian Numerals:** Always convert English digits to Persian digits (۰۱۲۳۴۵۶۷۸۹) for UI elements, dates, prices, countdown timers, and quantities.
  - _Example:_ `کد تایید ۵ رقمی را وارد کنید` (Not: `کد تایید 5 رقمی را وارد کنید`).
  - _Example:_ `۲۴ ساعت باقی‌مانده` (Not: `24 ساعت باقی مانده`).
- **Punctuation:** Use Persian comma (،) and question mark (؟). Do not leave a space before punctuation, but always leave one space after.

## 2. Voice and Tone (لحن و بیان)

Persian can easily slip into overly formal, bureaucratic language. We must balance respect with modern, conversational app experiences.

- **Respectful but Conversational (محترمانه و صمیمی):** Address the user in the second-person plural (شما) for politeness, but avoid administrative vocabulary.
  - _Bureaucratic (Avoid):_ `کاربر گرامی، لطفا نسبت به تکمیل پروفایل خود اقدام نمایید.`
  - _UX Optimized:_ `برای استفاده از امکانات بیشتر، پروفایل خود را کامل کنید.`
- **Active Voice over Passive Voice:** Passive voice in Persian often sounds clunky or robotic (e.g., heavily relying on the auxiliary verb "شد").
  - _English:_ Your password was changed successfully.
  - _Passive (Avoid):_ `رمز عبور شما با موفقیت تغییر داده شد.`
  - _Active (Prefer):_ `رمز عبور شما تغییر کرد.`

---

## 3. Translation vs. Localization (تله‌های ترجمه)

Direct translations from English UI patterns often fail in Persian. Context dictates the verb. Never map English words 1:1 to Persian without considering the user's immediate goal.

### 1. The "Submit" Trap

- **Problem:** Translating "Submit" to the generic "ارسال" (Send) or "تایید" (Confirm).
- _For a Marketplace / Auction:_ `ثبت آگهی` or `ثبت پیشنهاد`
- _For Account Creation:_ `ثبت‌نام` or `ایجاد حساب`
- _For Payment / Checkout:_ `پرداخت` or `نهایی‌کردن خرید`
- _For Feedback Form:_ `ارسال نظر`

### 2. The "OK / Confirm" Trap

- **Problem:** Using "باشه" (Too informal) or generic "تایید" for system dialogs.
- _Delete Modal:_ `حذف حذف شود` or `بله، حذف کن` (Action-specific).
- _Permission Dialog:_ `اجازه می‌دهم` or `متوجه شدم`.
- _Settings Saver:_ `ذخیره تغییرات`.

### 3. The "Cancel" Trap

- **Problem:** Confusing "انصراف" (Abandon process), "لغو" (Cancel ongoing status/subscription), and "بستن" (Dismiss UI).
- _Form / Modal Exit:_ `انصراف` or `بستن` (Leaves the form without saving).
- _Active Subscription or Order:_ `لغو اشتراک` or `لغو سفارش` (Cancels an active item).
- _Back Navigation:_ `بازگشت` (Returns to previous screen).

### 4. The "Delete" vs. "Remove" Trap

- **Problem:** Translating both to "حذف", blurring the line between permanent destruction vs. removing from a list.
- _Permanent Data Erasure (Delete):_ `حذف حساب` or `حذف دائمی` (High friction, red styling).
- _Remove item from Cart / Bookmark (Remove):_ `برداشتن از سبد` or `حذف از نشان‌شده‌ها`.

### 5. The "Sign In / Sign Up" Trap

- **Problem:** Using overly long phrasing like "ورود به سیستم / ثبت نام در سایت".
- _Primary Action Button:_ `ورود` / `ثبت‌نام`
- _Contextual CTA:_ `حساب کاربری دارید؟ ورود`

### 6. The "Retry / Try Again" Trap

- **Problem:** Using robotic phrases like "مجدداً تلاش فرمایید".
- _Network Error Action:_ `تلاش مجدد` or `سعی دوباره`
- _Payment Retry:_ `پرداخت مجدد`

### 7. The "Learn More / View Details" Trap

- **Problem:** Translating "Learn More" to literal "بیشتر بدانید" in space-constrained mobile banners.
- _Compact UI:_ `اطلاعات بیشتر` or `مشاهده جزئیات`
- _Link Label:_ `جزئیات بیشتر`

---

## 4. Mobile & RTL Constraints

- **String Expansion:** English phrases often expand by 20–30% when translated to Persian, and Persian font line-heights are typically taller.
- **Actionable Rule:** Keep mobile button labels to an absolute maximum of 2 to 3 words. Cut adjectives aggressively.
- **RTL Alignment:** Ensure directional indicators (arrows, chevrons) are flipped for Right-To-Left layouts. Contextual CTAs should place icons on the correct leading side.

---

## 5. Domain-Specific Examples

### Scenario A: Online Auction & Listing Marketplace (مزایده و فروشگاه)

- **CTA - Place Bid:**
  - _Bad:_ `پیشنهاد خود را ثبت کنید` (Too long for primary auction button).
  - _Good:_ `ثبت پیشنهاد` (Direct, concise).
- **Status - Outbid Notification:**
  - _Bad:_ `شما اوت‌بید شدید. پیشنهاد دیگری بدهید.` (Jargon/Transliteration).
  - _Good:_ `پیشنهاد بالاتری ثبت شد! برای ماندن در رقابت، پیشنهاد جدید بگذارید.` (Urgency + clear next action).
- **Validation Error - Bid Lower than Reserve:**
  - _Bad:_ `خطا: مبلغ نامعتبر است.` (Vague, unhelpful).
  - _Good:_ `مبلغ پیشنهادی باید حداقل ۱۰٪ از آخرین پیشنهاد بیشتر باشد.` (Clear criteria).
- **Empty State - Saved Listings:**
  - _Bad:_ `هیچ آگهی‌ای یافت نشد.` (Dead end).
  - _Good:_ `هنوز آگهی‌ای را نشان نکرده‌اید. آگهی‌های جالب را ذخیره کنید تا اینجا ببینید.` (Explains feature + CTA).

### Scenario B: Housing & Roommate Matching (هم‌اتاقی و مسکن)

- **Notification - New Match:**
  - _Bad:_ `یک هم‌اتاقی جدید برای شما پیدا شد.` (Passive voice).
  - _Good:_ `هم‌اتاقی جدید پیدا کردیم! پروفایل او را ببینید.` (Active, encouraging).
- **CTA - Request Contact:**
  - _Bad:_ `جهت برقراری ارتباط کلیک کنید` (Generic link text).
  - _Good:_ `درخواست گفتگو` or `ارسال پیام` (Specific action).
- **Error - No Matches with Current Filters:**
  - _Bad:_ `نتیجه‌ای یافت نشد.` (Unhelpful).
  - _Good:_ `با این فیلترها هم‌اتاقی پیدا نشد. تغییر محدوده قیمت یا محله را امتحان کنید.` (Provides immediate recovery step).
- **Onboarding Step - Verification:**
  - _Bad:_ `احراز هویت اجباری است.` (Aggressive tone).
  - _Good:_ `تایید هویت برای امنیت بیشتر. با تایید هویت، شانس پیدا کردن هم‌اتاقی را ۲ برابر کنید.` (Focuses on user benefit).

### Scenario C: Fintech, Digital Wallets & Payments (کیف پول و پرداخت)

- **CTA - Wallet Top-up:**
  - _Bad:_ `افزایش اعتبار حساب کاربری` (Too wordy).
  - _Good:_ `افزایش موجودی` (Standard fintech terminology).
- **Error - Declined Transaction:**
  - _Bad:_ `خطای ۴۰۲: پرداخت انجام نشد.` (Technical jargon).
  - _Good:_ `پرداخت ناموفق بود. موجودی کارت را بررسی کنید یا کارت دیگری بزنید.` (Explains failure without blame).
- **Success - Money Sent:**
  - _Bad:_ `عملیات با موفقیت انجام شد.` (Robotic).
  - _Good:_ `مبلغ انتقال یافت. رسید تراکنش آماده است.` (Clear confirmation).
- **Empty State - Transaction History:**
  - _Bad:_ `تراکنشی وجود ندارد.`
  - _Good:_ ` هنوز تراکنشی ثبت نکرده‌اید. اولین واریز یا برداشت شما اینجا نمایش داده می‌شود.`

### Scenario D: IoT & Smart Hardware Status (دستگاه‌های هوشمند)

- **System Banner - Offline Device:**
  - _Bad:_ `ارتباط قطعی است.`
  - _Good:_ `دستگاه آفلاین است. اتصال مودم یا Wi-Fi را بررسی کنید.` (Explains state + troubleshooting step).
- **Action - Pairing Device:**
  - _Bad:_ `شروع پروسه جفت‌سازی` (Heavy/Technical).
  - _Good:_ `اتصال به دستگاه` (Simple, direct).
- **Alert - Low Battery / Firmware Update:**
  - _Bad:_ `هشدار نرم‌افزار.`
  - _Good:_ `به‌روزرسانی جدید آماده است. برای کارکرد بهتر، دستگاه را به‌روز کنید.`

### Scenario E: SaaS, Account Settings & Forms (تنظیمات حساب و فرم‌ها)

- **Destructive Modal - Account Deletion:**
  - _Title:_ `حساب کاربری حذف شود؟`
  - _Body:_ `با حذف حساب، تمام اطلاعات و آگهی‌های فعال شما برای همیشه پاک می‌شوند.`
  - _Confirm Button:_ `بله، حذف شود` (Red button).
  - _Cancel Button:_ `انصراف` (Secondary style).
- **Inline Form Error - Password Strength:**
  - _Bad:_ `رمز عبور ضعیف است.`
  - _Good:_ `رمز عبور باید حداقل ۸ کاراکتر و شامل یک عدد باشد.` (Specific criteria).
