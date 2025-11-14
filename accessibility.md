# Background

To make the application fully accessibility-compliant, we are evaluating various tools in the market to identify the solution that best fits our technical ecosystem and product needs. Our goal is to adopt a tool that supports WCAG standards, integrates with our automation stack, and enables both automated and manual accessibility validation across mobile workflows.

# BrowserStack – App Accessibility Testing 

BrowserStack Accessibility Testing is a cloud-based platform that enables automated, assisted, and manual accessibility testing for both web and mobile applications. It supports real-device validation, including screen-reader testing on actual iOS and Android devices.

## Pros:

- **Automated & CI Integration:** Offers SDK/CLI tools that can be embedded into CI/CD pipelines to run accessibility checks early in development.

- **Assisted & Manual Testing:** Supports guided accessibility tests and allows real-device screen-reader validation using tools like TalkBack, VoiceOver, NVDA, etc., which helps verify true user experience.

- **Shift-Left Enablement:** Provides accessibility support across the development lifecycle — design guidelines, linters for developers, automation SDKs, and documentation for manual QA teams.

- **Reporting & Compliance:** Offers dashboards, issue summaries, and exportable reports aligned with WCAG 2.1 and other standards.

- **Fits Current Tech Stack:** Already part of our existing tool ecosystem, reducing onboarding friction and enabling quick adoption by engineering and QA teams.

- **Pricing:** Estimated to cost $7,500–$12,000 per year, depending on the plan.

- **Continuous Flow Scan (Supported):** The SDK can continuously monitor flows without requiring explicit invocation on each screen transition, improving testing efficiency.

## Cons:

- **Accuracy Variance:** As a broader testing platform (not exclusively accessibility-focused), rule accuracy and false-positive control may lag behind specialized vendors.

- **Cloud-Only Device Execution:** Since scans run on BrowserStack devices, results depend on network performance and device availability.

# axe DevTools Mobile – by Deque Systems

axe DevTools for Mobile is a dedicated mobile accessibility testing toolkit created by Deque — a recognized authority in the accessibility space and a contributor to the W3C accessibility guidelines. It enables automated accessibility scans for native and cross-platform mobile apps and integrates with Appium and other mobile testing frameworks.

## Pros:

- **Automated & CI Integration:** Offers SDKs and CLI tools to embed accessibility scans into pipelines such as Appium, Espresso, and XCUITest.

- **Shift-Left Enablement:** Strong tooling for designers, developers, and QA — including linters, best-practice analyzers, and documentation for manual testing workflows.

- **Reporting & Compliance:** Provides structured issue reports, dashboards, WCAG mapping, and export capabilities.

- **Accessibility Domain Experts:** Deque is a leader in the accessibility space; their team contributes to accessibility standards and works with major enterprises, especially in regulated industries (banking, healthcare, insurance).

- **High Accuracy, Low False Positives:** axe is widely regarded as one of the most accurate accessibility rule engines, minimizing noise in automation results.

- **Human-Verified Accessibility (Optional):** Deque provides services where trained accessibility testers can audit the app to ensure near-complete accessibility coverage — reducing internal qa, dev and design team workload.


## Cons:

- **No BrowserStack Device Support:** axe DevTools does not integrate with BrowserStack real devices and has no planned future support.

- **Requires Additional Device Infrastructure:** To fully run tests, we must rely on local devices, emulators, or supported cloud platforms like Sauce Labs — which may introduce additional operational or licensing costs.

- **Continuous Flow Scan (Not Supported):** The SDK requires manual invocation for each new screen render. Deque has confirmed they do not recommend or plan to support continuous automatic scanning across flows.

- **Integration Overhead:** Requires more setup and environment management (e.g., Appium driver wrapping, maintaining local devices/emulators).


## Other Observations

- **Driver Support with Appium:** axe provides Appium driver wrappers, allowing accessibility scans to run on local or on-premise test devices.

- **Potential Additional Costs:** If we need a device cloud alternative (like Sauce Labs), overall cost may increase significantly.

# General Notes (Applies to Both Tools)

- **No Tool Achieves 100% Automation:** Accessibility testing inherently requires manual verification (screen-reader navigation, focus order validation, color contrast in dynamic contexts, cognitive load checks, etc.). Both tools supplement but do not replace manual accessibility testing.

- **Complementary Use Case:** Automated scans catch structural & technical violations (labels, roles, touch targets, semantics). Manual usability and assistive technology validation must still be part of the process.

# Additional Points

**1. Choice Depends on Device Strategy**

If we rely heavily on BrowserStack real devices for functional & automation testing, BrowserStack’s accessibility solution integrates more naturally.

If we prefer local/on-premise or emulator-based testing, axe DevTools may provide deeper accuracy and tighter integration.

**2. Rule Engine Quality**

BrowserStack uses Spectra (their custom engine), whereas axe DevTools uses the axe-core rule engine, widely considered the industry standard and used by major companies like Microsoft, Google, and GitHub.

**3. Other Important parameters**

- If accessibility is a long-term strategic pillar (e.g., legal risk, large user base, public-facing product), axe DevTools offers stronger credibility and depth.

- If we need fast, scalable, easy-to-rollout checks across teams, BrowserStack is simpler and more cost-effective.

- BrowserStack is cheaper upfront and fits current infrastructure.

- axe DevTools may yield better long-term accessibility compliance due to expertise + accuracy + human audit options.