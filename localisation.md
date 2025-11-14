# Localisation and automation

This document list down a brief overview of the localisation the challenges it poses for automation testing and how can those challenges will be mitigated and what should be the automation approach (proposed). 

## Challenges and Impacts

Let's list down the challenges and it impact on the different areas of automation

| Area                         | Challenge                                                                      | Impact                                                                                              |
|------------------------------|--------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| Dynamic Language Content     | Text labels, buttons, and messages differ per locale (e.g., Login → Connexion) | Element locators based on visible text fail.                                                        |
| UI Expansion / Shrinkage     | Long or short strings change layout.                                           | Visual validations and hard coded element coordinates break.                                        |
| Right-to-Left (RTL) Layouts  | Arabic/Hebrew flip UI layout and scroll/swipe directions.                      | Scroll/swipe actions fail.                                                                          |
| Locale-specific Formats      | Dates, currency, decimals vary by region.                                      | Assertions fail if values are hardcoded.                                                            |
| Dynamic Fonts & Rendering    | Localized fonts may cause text clipping or truncation.                         | Visual checks and screenshots differ.                                                               |
| Frequent Translation Updates | Language files updated frequently by dev based on language requirements        | Have to refactor accordingly as per the new language updates                                        |
| Test Data Management         | May need to create test data in local languages to check support               | test data creation overhead increases additional libraries needed to create data in local languages |


# Best practices to address/mitigate these challenges

| # | Challenge Addressed                                                                                                                  | Category                                   | Best Practice                                                                                                                                                                                                 | Implementation Tip                                                                         |
|---|--------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------|
| 1 | Unstable element locators due to translated text /Dynamic Language Content                                                           | Locators                                   | Use stable identifiers — accessibility IDs or resource IDs, not visible text.                                                                                                                                 | Define testID or automationId attributes in dev code for automation use.                   |
| 2 | Assertion failures due to dynamic localized text / Dynamic Language Content / Locale-specific Formats / Frequent Translation Updates | Translated Strings                         | Keep expected localized strings in external JSON/YAML files (same as app i18n files). Automation should have access to these language files and libraries which are used so that tests can be made compatible | Example: /i18n/en.json, /i18n/fr.json, /i18n/ar.json                                       |
| 3 | Fragile locators in Page Objects                                                                                                     | Page Object Model (POM)                    | POMs reference element IDs, not text; load locale-specific expected strings dynamically from language files using the same library as used by the FE team.                                                    | expect(await getText()).toBe(localeData.loginSuccess // gives localised string same as FE) |
| 4 | Test Data Management                                                                                                                 | Test Data Management                       | Test data (test data base) on lower environments like preprod should not be restricted based on region/locale in order to avoid test data creation overheads                                                  |                                                                                            |
| 5 | visual validations / UI Expansion / Shrinkage                                                                                        | Visual Validation/UI Expansion / Shrinkage | Run visual tests on a single/default (most used) language                                                                                                                                                     |                                                                                            |
| 6 | Automation failures in right-to-left (RTL) languages                                                                                 | RTL Handling                               | Add conditional scroll/swipe logic based on locale direction.                                                                                                                                                 | Example: direction = locale === 'ar' ? 'right' : 'left'                                    |
| 7 | Long execution time across all locales                                                                                               | Smoke vs Full Suites                       | Run full regression for the default locale/most used language, smoke tests for others supported languages.                                                                                                    | Saves CI time while maintaining essential localization coverage.                           |


# Automation Requirements From Other Teams

From FE Team:

- Selectors should be language agnostic
- Accessibility IDs/automation ids should be there and should be language agnostic
- Localisation implementations or any changes should be shared with the automation team
    - Automation team should have access to language files which are used at the FE
    - Methods and utility fucntions or any changes in them should be informed


From BE Team:

- Test users should not be restricted based on the local or language change at least on the lower and preprod environments
- API's responses should be language agnostic all localisation logic should exist on front end