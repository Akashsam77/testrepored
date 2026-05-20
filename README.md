================================================================================
         RED TEAM DLP TEST REPORT — S3 DATA EXFILTRATION TEST TOOL
                       Walkthrough & Analysis Document
================================================================================

  Document Type   : Technical Walkthrough Report
  Classification  : CONFIDENTIAL — Authorized Personnel Only
  Test Tool       : S3_DLP_RedTeam_Tool.exe
  Test Type       : Data Loss Prevention (DLP) Validation
  Platform        : Windows 10/11 (GUI Executable)
  Test Environment: Isolated / Authorized Test Scope Only
  Prepared By     : Red Team
  Date            : May 2026

  WARNING: This document and the tools described herein are for AUTHORIZED
  security testing only. Unauthorized use is illegal and a violation of
  organizational policy.

================================================================================


--------------------------------------------------------------------------------
1. EXECUTIVE SUMMARY
--------------------------------------------------------------------------------

This report documents the walkthrough and usage of S3_DLP_RedTeam_Tool.exe —
a purpose-built Windows GUI application developed to perform authorized data
exfiltration simulation tests against AWS S3 storage. The tool is designed to
validate whether an organization's Data Loss Prevention (DLP) controls can
detect and block unauthorized data transfers to cloud storage.

The tool simulates 10 distinct exfiltration techniques, ranging from simple
file uploads to advanced bypass methods such as presigned URLs, encoded
payloads, and renamed file extensions. All tests use synthetic, clearly labeled
test data and target an authorized test S3 bucket.

Objective:
  Identify gaps in DLP coverage across CLI, GUI, and programmatic AWS S3 upload
  paths so that security controls can be tuned and hardened accordingly.


--------------------------------------------------------------------------------
2. TOOL OVERVIEW
--------------------------------------------------------------------------------

2.1  What is S3_DLP_RedTeam_Tool.exe?
--------------------------------------
S3_DLP_RedTeam_Tool.exe is a standalone Windows executable (no installation
required) that provides a graphical interface for running structured data
exfiltration tests against a target AWS S3 bucket. It is built using Python
(tkinter + boto3) and compiled with PyInstaller into a single portable file.

  Property        Details
  --------------- -------------------------------------------------------
  File Name       S3_DLP_RedTeam_Tool.exe
  Platform        Windows 10 / 11 (64-bit)
  Language        Python 3.12 (compiled via PyInstaller)
  Dependencies    Bundled — no Python, no AWS CLI, no extra install needed
  AWS SDK         boto3 (bundled inside exe)
  Test Methods    10 distinct exfiltration scenarios
  Output          On-screen results table + exportable CSV log


2.2  GUI Layout
----------------
The application window is split into two panels:

  LEFT PANEL
  ┌─────────────────────┬──────────────────────────────────────────────────┐
  │ Section             │ Purpose                                          │
  ├─────────────────────┼──────────────────────────────────────────────────┤
  │ AWS Credentials     │ Enter Access Key, Secret Key, Region             │
  │ Target Bucket       │ S3 bucket name and key prefix (folder)           │
  │ Test Data           │ Generate synthetic files or browse custom file   │
  │ Controls            │ Run / Stop / Cleanup / Export buttons            │
  └─────────────────────┴──────────────────────────────────────────────────┘

  RIGHT PANEL
  ┌─────────────────────┬──────────────────────────────────────────────────┐
  │ Section             │ Purpose                                          │
  ├─────────────────────┼──────────────────────────────────────────────────┤
  │ Test Scenarios      │ 10 checkboxes to select which tests to run       │
  │ Results Table       │ Live upload status + DLP alert column per test   │
  │ Activity Log        │ Timestamped console output for every action      │
  │ Progress Bar        │ Shows X / Y tests complete                       │
  └─────────────────────┴──────────────────────────────────────────────────┘


--------------------------------------------------------------------------------
3. STEP-BY-STEP WALKTHROUGH
--------------------------------------------------------------------------------

STEP 1 — Launch the Tool
--------------------------
Double-click S3_DLP_RedTeam_Tool.exe.

On first launch an Authorization Disclaimer dialog appears requiring the tester
to confirm:
  [x] Written Rules of Engagement authorization exists
  [x] Using an isolated test environment
  [x] All data is synthetic / non-sensitive
  [x] DLP monitoring team is on standby

Click "I Confirm — Proceed" to continue into the main application window.


STEP 2 — Enter AWS Credentials
--------------------------------
In the AWS Credentials panel (top-left):

  1. Enter the Access Key ID of the red team IAM test account
  2. Enter the Secret Access Key
  3. Set the Region (default: us-east-1)
  4. Click the [⚡ Connect to AWS] button

The connection status changes to GREEN (● Connected) on success.
Any credential error is displayed immediately in the Activity Log.

  NOTE: Minimum required IAM permissions:
    - s3:PutObject
    - s3:DeleteObject
    - s3:ListBucket
    (on the test bucket only)


STEP 3 — Configure Target Bucket
----------------------------------
In the Target Bucket panel:

  - Bucket Name : Enter the authorized test S3 bucket name
  - Key Prefix  : Folder path for test files (default: exfil/)


STEP 4 — Generate Synthetic Test Data
---------------------------------------
Click [🧪 Generate Synthetic Test Data].

Three files are created in a local temp folder (~/redteam_dlp_test/):

  File               Size    Contents / DLP Rule Targeted
  ------------------ ------- -----------------------------------------
  pii_data.csv       < 1 KB  Fake SSNs, credit cards, emails — PII rules
  confidential.txt   < 1 KB  CONFIDENTIAL keyword + budget data
  bulk_50mb.bin      50 MB   Random binary — volume threshold rules

  All files are clearly labeled: "RED TEAM DLP TEST ONLY"
  They contain NO real sensitive information.


STEP 5 — Select Test Scenarios
--------------------------------
Check or uncheck the 10 scenario checkboxes.
All 10 are selected by default.
Use [Select All] / [Deselect All] for quick toggling.


STEP 6 — Run the Tests
------------------------
Click [▶ Run Selected Tests].

The tool runs each test sequentially in a background thread:
  - Activity Log fills with timestamped entries per upload
  - Results Table updates after each test
  - Progress bar shows completion percentage
  - GUI stays fully responsive throughout

After all tests complete, the log shows: SESSION COMPLETE

The DLP team should then review their console for triggered alerts and fill
in the DLP Alert / Blocked columns in the results table.


STEP 7 — Export Results
-------------------------
Click [💾 Export Results CSV] to save the results table as a .csv file for
inclusion in the formal DLP test report.


STEP 8 — Cleanup
-----------------
Click [🗑 Cleanup All Artifacts] to:
  - Delete all uploaded S3 objects under the test prefix
  - Remove the local ~/redteam_dlp_test/ folder

  ⚠ IMPORTANT: Always run cleanup before ending the session to prevent
  accidental data exposure and avoid ongoing AWS storage costs.


--------------------------------------------------------------------------------
4. TEST SCENARIOS — DETAILED DESCRIPTION
--------------------------------------------------------------------------------

  ID    Name                   Method    Description & DLP Rule Targeted
  ----- ---------------------- --------- ----------------------------------------
  T-01  Single PII Upload      boto3     Uploads pii_data.csv directly.
                                         Targets PII content-inspection rules
                                         (SSN, credit card, email patterns).

  T-02  Bulk Directory Sync    boto3     Syncs entire test folder to S3.
                                         Targets multi-file and volume-based
                                         DLP policies.

  T-03  Large File 50 MB       boto3     Uploads a 50 MB binary file.
                                         Targets file-size threshold rules.

  T-04  Compressed TAR.GZ      boto3     Archives PII + confidential files into
                                         tar.gz before upload. Tests if DLP
                                         inspects inside compressed containers.

  T-05  Multipart Upload       boto3     Splits 50 MB into 5 MB chunks via S3
                                         multipart API. Tests if DLP reassembles
                                         and inspects chunked transfers.

  T-06  Presigned URL          urllib    Generates a presigned PUT URL then uploads
                                         via HTTP with no AWS credentials on the
                                         sender side. Tests unsigned-request DLP.

  T-07  Base64 Encoded         boto3     Encodes PII file as base64 before upload.
                                         Tests if DLP decodes and inspects
                                         encoded payloads.

  T-08  Renamed Extension      boto3     Renames pii_data.csv to vacation_photo.jpg.
                                         Tests if DLP uses content inspection vs
                                         extension-only matching.

  T-09  Password ZIP           boto3     Wraps PII in a standard ZIP file.
                                         Tests if DLP can inspect or block
                                         compressed containers.

  T-10  Confidential TXT       boto3     Uploads confidential.txt with
                                         CONFIDENTIAL keyword and strategy data.
                                         Targets keyword / regex DLP rules.


  Expected DLP Behaviour by Test:
  --------------------------------
  T-01, T-02, T-03, T-10  →  DLP SHOULD detect (content / volume / keyword rules)
  T-04, T-05, T-06        →  Potential bypass — verify manually
  T-07, T-08, T-09        →  Potential bypass — verify manually


--------------------------------------------------------------------------------
5. DLP VALIDATION CHECKLIST
--------------------------------------------------------------------------------

Fill in this table after running tests with the DLP team:

  Test  Scenario               Uploaded?  DLP Alert?  Alert Time  Blocked?  Notes
  ----- ---------------------- ---------- ----------- ----------- --------- ------
  T-01  Single PII CSV         [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-02  Bulk Sync              [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-03  Large File 50 MB       [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-04  TAR.GZ Archive         [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-05  Multipart Upload       [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-06  Presigned URL          [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-07  Base64 Encoded         [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-08  Renamed Extension      [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-09  Password ZIP           [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N
  T-10  Confidential TXT       [ ] Y [ ]N [ ] Y [ ]N  __________ [ ] Y [ ]N


--------------------------------------------------------------------------------
6. COMMON DLP GAPS & RECOMMENDATIONS
--------------------------------------------------------------------------------

  Gap                               Affected  Recommended Fix
  --------------------------------  --------  ----------------------------------
  Presigned URL bypasses DLP        T-06      Enable CASB with API-level S3
                                              visibility; inspect unsigned PUTs

  Compressed archives not           T-04,     Configure DLP to decompress and
  inspected                         T-09      inspect TAR.GZ and ZIP containers

  Base64 encoding bypasses          T-07      Enable content normalization /
  content scan                                decode-before-inspect in DLP engine

  Renamed extension not caught      T-08      Switch from extension-based to
                                              magic-byte / content-based file ID

  Chunked multipart not             T-05      Enable stream reassembly in proxy /
  reassembled                                 DLP for multipart S3 transfers

  Volume threshold too high         T-02,     Lower threshold; alert on any S3
                                    T-03      upload outside approved buckets


--------------------------------------------------------------------------------
7. POST-TEST CLEANUP PROCEDURE
--------------------------------------------------------------------------------

After every test session:

  1. Click [🗑 Cleanup All Artifacts] in the tool
     → Deletes all S3 objects under the test prefix
     → Removes local test files

  2. Verify S3 bucket is empty:
     AWS Console → S3 → check exfil/ prefix has no objects

  3. Confirm local folder ~/redteam_dlp_test/ has been removed

  4. Revoke or rotate the test IAM credentials if created for this exercise

  5. Provide exported CSV results to the DLP team for alert correlation

  6. Document any DLP gaps in the findings register

  ⚠ Never leave test data in S3 after a session. Even clearly labeled
  synthetic data must be removed to maintain a clean test environment.


--------------------------------------------------------------------------------
8. TOOL ARCHITECTURE NOTES
--------------------------------------------------------------------------------

8.1  How the EXE is Built
--------------------------
Written in Python 3.12, compiled to a standalone Windows executable using
PyInstaller (--onefile). All dependencies including boto3, botocore, and
tkinter are bundled inside the single .exe file.

Build command (run on Windows machine):

  python -m PyInstaller --onefile --windowed --name S3_DLP_RedTeam_Tool ^
    --collect-all boto3 --collect-all botocore s3_dlp_redteam_gui.py


8.2  Thread Safety
-------------------
All AWS operations (uploads, cleanup, connection tests) run on background
threads to keep the GUI responsive. UI updates (log, progress bar, results
table) are safely scheduled on the main thread using tkinter's
after(0, callback) mechanism — the correct approach for cross-thread GUI
updates in Python.


8.3  Data Handling
-------------------
The tool communicates only with AWS endpoints using credentials explicitly
provided by the tester. No data is sent anywhere else. All test data is
synthetic and generated locally on the test machine.


8.4  Key Files
---------------
  s3_dlp_redteam_gui.py  →  Python source code
  BUILD_EXE.bat          →  Windows build script (run once to compile .exe)
  S3_DLP_RedTeam_Tool.exe→  Compiled standalone executable (output)


--------------------------------------------------------------------------------
9. TEST SIGN-OFF
--------------------------------------------------------------------------------

  Role                    Name              Date          Signature
  ----------------------  ----------------  ------------  --------------------
  Red Team Lead           _______________   ___________   ____________________
  DLP Engineer            _______________   ___________   ____________________
  Security Manager        _______________   ___________   ____________________
  Test Environment Owner  _______________   ___________   ____________________


================================================================================
  CONFIDENTIAL — Authorized Internal Use Only — Do Not Distribute
  This report is restricted to personnel directly involved in the DLP
  validation exercise.
================================================================================
