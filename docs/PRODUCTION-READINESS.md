# Production acceptance — 2026-10-06

## Decision

Update 2026-10-07: the long-description correction now passes in a separate laboratory copy, including the signed reduced runtime. See SALE-LAYOUT-FIX.md. Original ERP reports remain unchanged; production release gates below still apply. The legacy editor explicitly uses PERSISTENCE_NEVER, so tab restoration is an inherited unsupported behavior, not a newly established regression.

The laboratory passes are evidence for the updated designer and isolated report runtime. They do **not** certify the current delivered ERP jar or physical printing for production. Do not deploy the laboratory ERP jars. No protected ERP file is changed by this task.

## Confirmed

- Apache NetBeans 31 / JDK 21: final plugin Maven build succeeds; 9 automated tests pass, including compilation of 215 copied ERP templates. Earlier JDK 25 build also passed.
- Native visual editor: sale opens with the laboratory application jar in iReport Classpath; `Report Inspector > report name > Compile Report` generates its Jasper binary. `Tools > Options > iReport` and its Classpath panel open without closing the IDE.
- The native compiled sale binary runs with the coherent Java 17 laboratory runtime, including the real `Util.MetodosUteis` method. Cash and product parents also render their subreports.
- Native Jasper-to-JRXML conversion and complete structural comparison previously passed for all six templates; see ROUNDTRIP-VALIDATION.md.
- Synthetic sale data: zero rows gives zero pages (existing template behavior); one long item renders one continuous page; 80 items render one continuous page; explicitly enabling pagination gives eight pages. PDF content checks retain the first/last items and accented text.
- Offline Sicoob boleto rendering, barcode rendering and PDF merging pass in the full coherent jar.
- A temporary self-signed laboratory certificate signs the full jar; `jarsigner -verify` succeeds and the signed jar runs all three report parents. Expected self-signed, expiry and timestamp warnings do not constitute production certificate validation. Production keystore and TSA were not used.
- The final round-3 reduced jar is also signed and verified with the same temporary certificate. Its signed form passes all report/layout data checks and offline boleto/barcode/PDF-merge checks (`small-signed-runtime.log`, `small-signed-boleto.log`, `small-signing-verify.log`).
- Packaging audit verifies the application Main-Class is preserved, ZIP names are unique, and representative JasperReports, Groovy and OpenPDF classes match official dependencies.

## Material findings

1. **Layout fails for very long descriptions.** Rendered sale PDFs show text overlapping quantity/price. The existing detail elements use fixed vertical positions; compilation and text extraction alone do not detect this. Evidence: `runtime-candidate-02/layout-pdf/sale-layout-1.png` and `sale-paginated-last.png`. The original report remains unchanged. Fixing this report requires a separately authorized change to its layout, followed by visual and printer acceptance.
2. The delivered ERP jars embed older JasperReports/PDF classes from java-boletos and need coherent assembly before consuming newly compiled reports. Installing the editor plugin alone does not replace the ERP runtime. See FINAL-JAR-VALIDATION.md.
3. Existing ProGuard rules remove reflective/dynamic dependencies. Round 1 loses the Hibernate Validator generated JBoss logging implementation. Round 2 restores validation but loses OpenPDF's reflected `Jpeg(Image)` constructor. The laboratory rules preserve the relevant validation/logging/EL and PDF packages. **Round 3 passes** all three parent reports, native sale expression access, empty/long/80-item sale cases, explicit eight-page pagination, offline Sicoob barcode rendering and two-page PDF merging. Evidence: `small-round3-runtime.log` and `small-round3-boleto.log`. Full jar: 265,368,470 bytes; reduced round-3 jar: 197,657,115 bytes. This validates those paths, not every ERP function.
4. NetBeans logs still warn that the legacy multi-view description is not serializable. Opening/compilation passes; restoration of editor tabs across restarts is not approved by this run. The IDE Janitor also attempted to clean older laboratory profiles. Keep laboratory evidence outside disposable IDE profiles.

## Protection and artifact

Final SHA-256 check: all 538 protected report/POM files remain identical. Seven unrelated ERP source/form/test files differ from the earlier 3,102-file baseline during concurrent work. This task performed no ERP writes and did not revert those changes. Evidence: `runtime-candidate-02/protected-integrity-final.json`.

Final plugin artifact is staged at `laboratorio/artifacts-production-candidate/ireport-designer-6.0-SNAPSHOT.nbm`, with SHA-256 beside it. It has not been installed in the user's daily ERP NetBeans profile.

## Release gates

- Resolve long-description layout with authorized changes or explicitly constrain supported input; verify no overlap.
- Rebuild the actual ERP with coherent assembly and validated shrinking rules in an authorized branch/copy, and run its application acceptance checks. Laboratory repacking is not a substitute for that build.
- Verify the actual production signing certificate and complete signing pipeline.
- Print representative sales on the affected user's printer, confirming paper length, margins, page footer and cutter behavior. This task does not have that printer or its driver configuration.
- Verify restart/tab restoration if it is required in the designer workflow.

Rollback: preserve the old plugin backup and old report/ERP delivery; never replace them with unsigned laboratory candidates. The plugin backup is in `laboratorio/backup-plugin-5.5.0`.
