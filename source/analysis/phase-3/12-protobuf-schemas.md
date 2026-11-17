# 12 — Protobuf schema reconstruction (wire-format interface documentation)

**Phase:** 3 (2026-11-16). **Nature of this document:** a *descriptive reconstruction* of the protobuf/gRPC surface of DigiExam Windows 26.4.7, produced exclusively from embedded strings and generated code descriptors recovered from the signed artifacts. This is **not** the original `.proto` source; no repository code, no network traffic, and no execution was involved (static, read-only analysis). Labels: **VERIFIED** (string/descriptor present at the cited location), **STRONGLY INFERRED** (grouping/attribution deduced from adjacency or convention), **UNVERIFIED**.

Two evidence classes with different fidelity:

1. **Frontend protobuf-es descriptors** (generated client compiled into `index-COO2fUbz.js`): full field *numbers*, *names*, *kinds* (scalar/enum/message/map), repetition, and scalar types survive in the minified bundle (`proto3.util.newFieldList` arrays). These allow a near-complete wire-schema reconstruction — with the caveat that it is the *client-side view* of the API.
2. **Rust-side reflection strings** (prost `Name`/`Debug` format strings and serde key strings in `digiexam.exe` / `service.exe` / `toolbox-ipc-proxy.exe`): field *names* only (no numbers, no types). Used to corroborate the frontend schemas and to document native-only structures (IPC payloads, snapshot JSON).

## 1. Sources

| Source | Artifact | Contributes | Evidence form |
|---|---|---|---|
| Frontend bundle | `source/recovered-assets/frontend/assets/index-COO2fUbz.js` | 8 services / 199 RPCs; 489 message descriptors; 37 enums | char offsets (`@NNN`) |
| digiexam.exe strings | `source/recovered-assets/strings/strings_digiexam.txt` | tonic method paths, native serde/prost vocabulary | line no. + raw byte offset in `D:\Digiexam\digiexam.exe` |
| service.exe strings | `.../strings_service.txt` | same for the lockdown Windows service | line no. + raw offset |
| toolbox proxy strings | `.../strings_toolbox.txt` | *older* schema iteration of the lockdown IPC vocabulary | line no. + raw offset |
| Binaries (offset pinning, read-only) | `D:\Digiexam\{digiexam,service,toolbox-ipc-proxy}.exe` | raw file offsets for key strings | `0x…` offsets below |

Method-path convention (fully-qualified `/package.Service/Method`) follows the official gRPC HTTP/2 protocol; see [`10-backend-protocols.md`](../phase-2/10-backend-protocols.md) for the transport analysis.

## 2. Service & method inventory

### 2.1 Summary — frontend generated service definitions (VERIFIED, bundle char offsets)

| Service (fully-qualified) | Methods | Evidence (bundle offset) |
|---|---|---|
| `assignment.service.v1.AssignmentService` | 36 | typeName @1659209; inline I/O + `MethodKind` per method |
| `directorysync.service.v1.DirectorySyncService` | 12 | first method @1714073 |
| `event.service.v1.EventService` | 4 | @1741178 |
| `iam.service.v1.IAMService` | 71 | @1990748 |
| `intserver.service.v1.IntServerService` | 1 | @2003407 |
| `lis.service.v1.LearningIntegrationService` | 41 | @2193660 |
| `lis.service.v1.UserService` | 3 | @2211315 |
| `lockdown.service.v1.LockdownService` | 31 | @2343205 |
| **Total** | **199** | — |

Two generation styles coexist: `AssignmentService` entries carry `{ name, I, O, kind }` inline; the other services carry `{ kind }` only (I/O message pairing follows the `*Request`/`*Response` naming convention — STRONGLY INFERRED for those). `assignment.service.v1.EmptyMessage` (@1672545) is a shared empty type.

### 2.2 Frontend RPC method paths (VERIFIED, per-method char offsets)

Kind per descriptor (`MethodKind.Unary` default where folded).

### assignment.service.v1.AssignmentService -- 36 methods

```text
LoginWithLegacyToken (Unary) [LoginWithLegacyTokenRequest -> LoginWithLegacyTokenResponse] @1659368
StandardLogin (Unary) [StandardLoginRequest -> StandardLoginResponse] @1659737
RefreshToken (Unary) [RefreshTokenRequest -> RefreshTokenResponse] @1660064
GetImmersiveReaderToken (Unary) [GetImmersiveReaderTokenRequest -> GetImmersiveReaderTokenResponse] @1660399
SelfSignUp (Unary) [SelfSignUpRequest -> SelfSignUpResponse] @1660778
SendLoginCode (Unary) [SendLoginCodeRequest -> SendLoginCodeResponse] @1661096
GenerateLoginCode (Unary) [GenerateLoginCodeRequest -> GenerateLoginCodeResponse] @1661449
GenerateLoginCodeForStudent (Unary) [GenerateLoginCodeForStudentRequest -> GenerateLoginCodeForStudentResponse] @1661833
GetLoginCode (Unary) [GetLoginCodeRequest -> GetLoginCodeResponse] @1662229
LoginWithCode (Unary) [LoginWithCodeRequest -> LoginWithCodeResponse] @1662567
GetActiveExams (Unary) [GetActiveExamsRequest -> GetActiveExamsResponse] @1662905
GetExamDetails (Unary) [GetExamDetailsRequest -> GetExamDetailsResponse] @1663238
SearchExamDetails (Unary) [SearchExamDetailsRequest -> SearchExamDetailsResponse] @1663574
StartOnlineExam (Unary) [StartOnlineExamRequest -> StartOnlineExamResponse] @1663931
StartUSBExam (Unary) [StartUSBExamRequest -> StartUSBExamResponse] @1664271
GetQuestions (Unary) [GetQuestionsRequest -> GetQuestionsResponse] @1664581
GetAnswers (Unary) [GetAnswersRequest -> GetAnswersResponse] @1664911
SaveMultipleChoiceAnswer (Unary) [SaveMultipleChoiceAnswerRequest -> SaveMultipleChoiceAnswerResponse] @1665241
SaveSingleChoiceAnswer (Unary) [SaveSingleChoiceAnswerRequest -> SaveSingleChoiceAnswerResponse] @1665637
SaveTextualAnswer (ClientStreaming) [SaveTextualAnswerRequest -> SaveTextualAnswerResponse] @1665998
SaveTextualAnswerUnary (Unary) [SaveTextualAnswerUnaryRequest -> SaveTextualAnswerUnaryResponse] @1666352
SaveWordGapAnswer (Unary) [SaveWordGapAnswerRequest -> SaveWordGapAnswerResponse] @1666728
HandInExam (Unary) [HandInExamRequest -> HandInExamResponse] @1667057
UploadFile (ClientStreaming) [UploadFileRequest -> UploadFileResponse] @1667347
DownloadFile (ServerStreaming) [DownloadFileRequest -> DownloadFileResponse] @1667649
SendLog (Unary) [SendLogRequest -> SendLogResponse] @1667900
StreamLog (ClientStreaming) [StreamLogRequest -> StreamLogResponse] @1668116
CheckDeprecation (Unary) [CheckDeprecationRequest -> CheckDeprecationResponse] @1668438
GenerateProctoringToken (Unary) [GenerateProctoringTokenRequest -> GenerateProctoringTokenResponse] @1668944
GetDefaultProctoringSetting (Unary) [GetDefaultProctoringSettingRequest -> GetDefaultProctoringSettingResponse] @1669314
CheckProctoringLicenses (Unary) [CheckProctoringLicensesRequest -> CheckProctoringLicensesResponse] @1669724
GetAnswerVersions (Unary) [GetAnswerVersionsRequest -> GetAnswerVersionsResponse] @1670118
RestoreExam (Unary) [RestoreExamRequest -> RestoreExamResponse] @1670500
CreateLockdownSystemSnapshotUpload (Unary) [CreateLockdownSystemSnapshotUploadRequest -> CreateLockdownSystemSnapshotUploadResponse] @1670829
SetConnectivityStatus (Unary) [SetConnectivityStatusRequest -> SetConnectivityStatusResponse] @1671256
GetConnectivityStatus (Unary) [GetConnectivityStatusRequest -> GetConnectivityStatusResponse] @1671625
```

### directorysync.service.v1.DirectorySyncService -- 12 methods

```text
CreateConnection (Unary) @1714073
GetOrganizationGroups (Unary) @1714343
CreateSyncGroups (Unary) @1714609
SyncOrganization (Unary) @1714842
SyncAllOrganizations (Unary) @1715084
GetSyncTask (Unary) @1715320
GetUsersByGroups (Unary) @1715560
GetUsersWithoutRoleTemplates (Unary) @1715833
MapUsersToRoleTemplates (Unary) @1716098
PropagateAllUsers (Unary) @1716396
GetIAMUsersForIntegration (Unary) @1716564
DisableIntegration (Unary) @1716748
```

### event.service.v1.EventService -- 4 methods

```text
SendClientLog (Unary) @1741178
StreamClientLog (ClientStreaming) @1741392
SendClientLogSession (Unary) @1741631
SendEvent (Unary) @1741848
```

### iam.service.v1.IAMService -- 71 methods

```text
CreateOrg (Unary) @1990748
GetOrgByID (Unary) @1990880
GetOrgsWithFilter (Unary) @1991014
UpdateOrg (Unary) @1991162
ActivateOrg (Unary) @1991294
DeactivateOrg (Unary) @1991430
DeleteOrg (Unary) @1991570
CreateFeature (Unary) @1991725
AddFeatureToOrgs (Unary) @1991865
GetOrgFeatures (Unary) @1992011
GetAllOrgsWithFeature (Unary) @1992153
RemoveFeatureFromOrgs (Unary) @1992309
DeleteFeature (Unary) @1992465
CreateUser (Unary) @1992635
CreateUsers (Unary) @1992769
GetUserById (Unary) @1992905
GetUsersByIds (Unary) @1993041
GetUsersWithFilter (Unary) @1993181
GetUsersByEmails (Unary) @1993331
UpdateUser (Unary) @1993477
ActivateUser (Unary) @1993611
DeactivateUser (Unary) @1993749
DeleteUser (Unary) @1993891
RegisterIdp (Unary) @1994043
GetIdpById (Unary) @1994179
GetIdpsWithFilter (Unary) @1994313
EnableIdp (Unary) @1994461
DisableIdp (Unary) @1994593
DeleteIdp (Unary) @1994727
LinkIdpToUser (Unary) @1994859
CreateLoginLink (Unary) @1995021
CollectTokens (Unary) @1995165
CreateLogoutLink (Unary) @1995305
CreateZitadelAuthLoginLink (Unary) @1995451
UpdatePasswordPolicy (Unary) @1995640
UpdateMfaPolicy (Unary) @1995794
UpdateLockoutPolicy (Unary) @1995938
UpdateLoginLifetimePolicy (Unary) @1996090
ResetOrgPoliciesToDefault (Unary) @1996254
GetDefaultPolicySet (Unary) @1996418
GetPolicySetByOrgId (Unary) @1996570
UpdateDefaultPasswordPolicy (Unary) @1996722
UpdateDefaultMfaPolicy (Unary) @1996890
UpdateDefaultLockoutPolicy (Unary) @1997048
UpdateDefaultLoginLifetimePolicy (Unary) @1997214
GetOrgAppsByOrgId (Unary) @1997410
ActivateCanvas (Unary) @1997558
DeactivateCanvas (Unary) @1997700
SetOrgAppLink (Unary) @1997846
GetGrantedAppsByOrgId (Unary) @1997986
GrantAppToOrg (Unary) @1998142
RevokeAppFromOrg (Unary) @1998282
GetAllPermissions (Unary) @1998490
GetAllDigiexamInternalPermissions (Unary) @1998638
CreateCustomTemplate (Unary) @1998859
GetCustomTemplatesByOrgID (Unary) @1999013
UpdateCustomTemplate (Unary) @1999177
DeleteCustomTemplate (Unary) @1999331
CreateGlobalTemplate (Unary) @1999526
GetGlobalTemplates (Unary) @1999680
UpdateGlobalTemplate (Unary) @1999830
DeleteGlobalTemplate (Unary) @1999984
CreateTemplateMapping (Unary) @2000189
GetTemplateMappingsByUserIDs (Unary) @2000345
UpdateTemplateMapping (Unary) @2000515
UpdateTemplateMappingByEmail (Unary) @2000671
GetUserIDsByPermission (Unary) @2000893
CreateDigiexamInternalTemplate (Unary) @2001098
UpdateDigiexamInternalTemplate (Unary) @2001272
CreateDigiexamInternalTemplateMapping (Unary) @2001446
UpdateDigiexamInternalTemplateMapping (Unary) @2001634
```

### intserver.service.v1.IntServerService -- 1 methods

```text
Ping (Unary) @2003407
```

### lis.service.v1.LearningIntegrationService -- 41 methods

```text
CreateConnection (Unary) @2193660
DeleteConnection (Unary) @2193867
ListConnections (Unary) @2194091
StartConnectionReauthorization (Unary) @2194391
RegisterDigiexamIAM (Unary) @2194662
UnregisterDigiexamIAM (Unary) @2194913
GetProviderUserID (Unary) @2195171
SyncProviderUsers (Unary) @2195335
ListAuthenticationProviders (Unary) @2195499
SetFallbackAuthenticationProvider (Unary) @2195683
SetupAuthenticationProvider (Unary) @2195879
GetProviderRoles (Unary) @2196063
EnsureResource (Unary) @2196321
ResolveExternal (Unary) @2196545
GetResource (Unary) @2196751
ListResourcesByOrg (Unary) @2196963
UpsertPolicy (Unary) @2197181
DeletePolicy (Unary) @2197376
GetUserActivities (Unary) @2197593
GetCoursesByUserID (Unary) @2197827
GetActivitiesWithFilter (Unary) @2198065
GetUsersByCourse (Unary) @2198304
GetGroupsByCourse (Unary) @2198522
GetSectionsByCourse (Unary) @2198744
GetUsersByGroupOrSection (Unary) @2198993
GetScheduledAccessSetupsByActivity (Unary) @2199268
GetCurrentLISControlledActivitiesByCourse (Unary) @2199555
GetStudentAssessmentAttempts (Unary) @2199881
UpdateResourceLockForUser (Unary) @2200132
GetResourceLockForUsers (Unary) @2200373
GetWorkspaceActivities (Unary) @2200619
CreateScheduledAccessSetup (Unary) @2200849
UpdateScheduledAccessSetup (Unary) @2201093
DeleteScheduledAccessSetup (Unary) @2201346
CancelScheduledAccessSetup (Unary) @2201582
HandInExam (Unary) @2201831
ReleaseUser (Unary) @2202029
ExtendScheduledAccessUserTime (Unary) @2202229
GetSessionActivities (Unary) @2202471
SendSessionActivity (Unary) @2202697
GetUsersByAssignment (Unary) @2202936
```

### lis.service.v1.UserService -- 3 methods

```text
CreateUser (Unary) @2211315
UpdateUser (Unary) @2211450
RollbackLogins (Unary) @2211585
```

### lockdown.service.v1.LockdownService -- 31 methods

```text
CreateOperationDefinition (Unary) @2343205
GetAllOperationDefinitions (Unary) @2343379
UpdateOperationDefinition (Unary) @2343555
DeleteOperationDefinition (Unary) @2343729
CreateOperatingSystem (Unary) @2343920
GetAllOperatingSystems (Unary) @2344086
CreateOperatingSystemArchitecture (Unary) @2344275
GetAllOperatingSystemArchitectures (Unary) @2344465
CreateDefaultOperation (Unary) @2344689
GetAllDefaultOperations (Unary) @2344857
OverrideOperation (Unary) @2345051
ArchiveOperation (Unary) @2345209
RestoreArchivedOperation (Unary) @2345365
GetManyOperationsWithFilter (Unary) @2345578
GetManyOrgOperationsWithFilter (Unary) @2345756
GetAllApplicableOperations (Unary) @2345940
GetLockdownConfig (Unary) @2346116
GetLockdownConfigOffline (Unary) @2346274
UpdateOperation (Unary) @2346446
DeleteOperation (Unary) @2346600
CreateOperationPlatform (Unary) @2346787
GetAllOperationPlatforms (Unary) @2346957
DeleteOperationPlatform (Unary) @2347129
AssignOperationPlatformsToOperation (Unary) @2347299
CreateExecutionResult (Unary) @2347524
GetExecutionResultsByIDs (Unary) @2347690
GetAggregatedExecutionResultInfo (Unary) @2347862
GetManyExecutionResultsWithFilter (Unary) @2348050
GetExecutionResultLogsInternal (Unary) @2348240
GetExecutionResultLogsPublic (Unary) @2348424
DeleteExecutionResult (Unary) @2348604
```

### 2.3 Rust-side tonic surface (VERIFIED, raw offsets in signed binaries)

Generated tonic client modules (prost/tonic codegen path strings — VERIFIED):

| Module path string | digiexam.exe | service.exe |
|---|---|---|
| `native\libs\protocol-client\src\lockdown.service.v1.tonic.rs` | @0x01AF5ECB | @0x00C612D0 |
| `native\libs\protocol-client\src\assignment.service.v1.tonic.rs` | present (strings) | present (strings) |
| `native\libs\protocol-client\src\event.service.v1.tonic.rs` | present (strings) | not observed |
| `native\libs\protocol-client\src\intserver.service.v1.tonic.rs` | present (strings) | not observed |
| `opentelemetry-proto-0.30.0\src\proto\tonic\…` (logs + trace) | present | present |

Observed tonic method-path strings (fully-qualified `/package.Service/Method`) — VERIFIED with first-observed raw offsets:

| Method path | digiexam.exe | service.exe |
|---|---|---|
| `/lockdown.service.v1.LockdownService/GetLockdownConfig` | 0x01AF5F00 | 0x00C61308 |
| `/assignment.service.v1.AssignmentService/RefreshToken` | 0x01AF5DD8 | — (module present) |
| `/assignment.service.v1.AssignmentService/SaveTextualAnswer` | 0x01B2EF48 | — |
| `/assignment.service.v1.AssignmentService/SetConnectivityStatus` | 0x01AF5E58 | — |
| `/assignment.service.v1.AssignmentService/CheckDeprecation` | 0x01B2EEE0 | — |
| `/assignment.service.v1.AssignmentService/CreateLockdownSystemSnapshotUpload` | 0x01FE81E8 | 0x00CA2C88 |
| `/event.service.v1.EventService/StreamClientLog` | 0x01AE5108 | — |
| `/intserver.service.v1.IntServerService/Ping` | 0x01B2F0A8 | — |
| `/opentelemetry.proto.collector.logs.v1.LogsService/Export` | present | present |
| `/opentelemetry.proto.collector.trace.v1.TraceService/Export` | present | present |

The Rust client therefore exercises a *subset* of the 199-RPC frontend surface (plus OTLP). The Rust-side `RefreshTokenResponse` serde string cluster (`RefreshTokenResponseaccess_tokenaccess_token_expires_at`, digiexam.exe @0x01E7DA48, @0x01FD0A20) matches the frontend `RefreshTokenResponse` fields 2 and 4 — cross-stack corroboration.

## 3. Reconstructed message schemas — frontend protobuf-es descriptors

**How to read:** reconstructed proto-style declarations grouped by package. Every declaration is derived from a `__publicField(_, "typeName", "<fqname>")` + `proto3.util.newFieldList([…])` descriptor pair in the bundle; the `@NNN` annotation after each message/enum is the char offset of its `typeName` literal. Field numbers, types, `// repeated` markers and `map<K,V>` come verbatim from the descriptor (`no`, `kind`, `T`, `repeated`). Fully-qualified type references are kept to preserve package attribution. `{ }` = descriptor declares zero fields (VERIFIED empty message). `oneof`: `lockdown.parameter.v1.ParameterValue` wraps its `*_val` fields in `oneof value` (descriptor comment `@generated from oneof lockdown.parameter.v1.ParameterValue.value`).

Widely-used well-known types: `google.protobuf.Timestamp`, `google.protobuf.Duration`, `google.protobuf.Struct`.


### assignment.answer.v1 -- 18 messages, 3 enums

```proto
enum AnswerBlockType {   ANSWER_BLOCK_TYPE_UNSPECIFIED = 0;   ANSWER_BLOCK_TYPE_HTML = 1;   ANSWER_BLOCK_TYPE_LATEX = 2;   ANSWER_BLOCK_TYPE_SVG = 3; }  // assignment.answer.v1.AnswerBlockType @1497136
enum AnswerAttachmentType {   ANSWER_ATTACHMENT_TYPE_UNSPECIFIED = 0;   ANSWER_ATTACHMENT_TYPE_IMAGE = 1;   ANSWER_ATTACHMENT_TYPE_MATH = 2; }  // assignment.answer.v1.AnswerAttachmentType @1507559
enum AnswerState {   ANSWER_STATE_UNSPECIFIED = 0;   ANSWER_STATE_EMPTY = 1;   ANSWER_STATE_DRAFT = 2;   ANSWER_STATE_FINAL = 3;   ANSWER_STATE_BLANK = 4; }  // assignment.answer.v1.AnswerState @1508182
message AnswerBlock {  // @1498471
  int32 block_id = 1;
  assignment.answer.v1.AnswerBlockType type = 2;
  string body = 3;
  int32 word_count = 4;
}
message GetAnswersRequest {  // @1509556
  int64 student_id = 1;
  int64 exam_id = 2;
}
message GetAnswersResponse {  // @1511612
  map<int64, assignment.answer.v1.TextualAnswer> textual = 1;
  map<int64, assignment.answer.v1.SingleChoiceAnswer> single_choice = 2;
  map<int64, assignment.answer.v1.MultipleChoiceAnswer> multiple_choice = 3;
  map<int64, assignment.answer.v1.WordGapAnswer> word_gap = 4;
}
message MultipleChoiceAnswer {  // @1500288
  int64 question_id = 1;
  int64 selected_options = 2;  // repeated
  google.protobuf.Timestamp updated_at = 3;
}
message SaveMultipleChoiceAnswerRequest {  // @1513645
  int64 exam_id = 1;
  int64 student_id = 2;
  assignment.answer.v1.MultipleChoiceAnswer answer = 3;
}
message SaveMultipleChoiceAnswerResponse { }  // empty @1514939
message SaveSingleChoiceAnswerRequest {  // @1516622
  int64 exam_id = 1;
  int64 student_id = 2;
  assignment.answer.v1.SingleChoiceAnswer answer = 3;
}
message SaveSingleChoiceAnswerResponse { }  // empty @1517890
message SaveTextualAnswerRequest {  // @1519493
  int64 exam_id = 1;
  int64 student_id = 2;
  assignment.answer.v1.TextualAnswer answer = 3;
}
message SaveTextualAnswerResponse { }  // empty @1520696
message SaveTextualAnswerUnaryRequest {  // @1522319
  int64 exam_id = 1;
  int64 student_id = 2;
  assignment.answer.v1.TextualAnswer answer = 3;
}
message SaveTextualAnswerUnaryResponse { }  // empty @1523582
message SaveWordGapAnswerRequest {  // @1525222
  int64 exam_id = 1;
  int64 student_id = 2;
  assignment.answer.v1.WordGapAnswer answer = 3;
}
message SaveWordGapAnswerResponse { }  // empty @1526425
message SingleChoiceAnswer {  // @1501995
  int64 question_id = 1;
  int64 selected_option = 2;
  google.protobuf.Timestamp updated_at = 3;
}
message TextualAnswer {  // @1503637
  int64 question_id = 1;
  assignment.answer.v1.AnswerBlock answer_blocks = 2;  // repeated
  int32 version = 3;
  google.protobuf.Timestamp updated_at = 4;
}
message WordGapAnswer {  // @1505407
  int64 question_id = 1;
  assignment.answer.v1.WordGapEntry gap_entries = 2;  // repeated
  google.protobuf.Timestamp updated_at = 3;
}
message WordGapEntry {  // @1506815
  string id = 1;
  string phrase = 2;
}
```

### assignment.answerversion.v1 -- 5 messages, 0 enums

```proto
message AnswerVersion {  // @1527688
  int32 version = 1;
  string created_at = 2;
  int32 state = 3;
  int32 size = 4;
}
message GetAnswerVersionsRequest {  // @1529212
  int64 exam_id = 1;
  int64 student_id = 2;
}
message GetAnswerVersionsResponse {  // @1530504
  assignment.answerversion.v1.AnswerVersion items = 1;  // repeated
}
message RestoreExamRequest {  // @1531824
  int64 exam_id = 1;
  int64 student_id = 2;
  int32 version = 3;
}
message RestoreExamResponse {  // @1533191
  string uri = 1;
  string url = 2;
}
```

### assignment.auth.v1 -- 20 messages, 0 enums

```proto
message GenerateLoginCodeForStudentRequest {  // @1536066
  int64 student_id = 1;
}
message GenerateLoginCodeForStudentResponse {  // @1537548
  string code = 1;
  google.protobuf.Timestamp expires_at = 2;
}
message GenerateLoginCodeRequest { }  // empty @1538683
message GenerateLoginCodeResponse {  // @1539935
  string code = 1;
  google.protobuf.Timestamp expires_at = 2;
}
message GetImmersiveReaderTokenRequest { }  // empty @1541078
message GetImmersiveReaderTokenResponse {  // @1542495
  string access_token = 1;
  int64 expires_in = 2;
}
message GetLoginCodeRequest { }  // empty @1543618
message GetLoginCodeResponse {  // @1544808
  string code = 1;
  google.protobuf.Timestamp expires_at = 2;
}
message LoginWithCodeRequest {  // @1546161
  string email = 1;
  string code = 2;
}
message LoginWithCodeResponse {  // @1548310
  assignment.auth.v1.User user = 1;
  string access_token = 2;
  string refresh_token = 3;
  google.protobuf.Timestamp access_token_expires_at = 4;
  google.protobuf.Timestamp refresh_token_expires_at = 5;
}
message LoginWithLegacyTokenRequest {  // @1549881
  string legacy_token = 1;
}
message LoginWithLegacyTokenResponse {  // @1552026
  assignment.auth.v1.User user = 1;
  string access_token = 2;
  string refresh_token = 3;
  google.protobuf.Timestamp access_token_expires_at = 4;
  google.protobuf.Timestamp refresh_token_expires_at = 5;
}
message RefreshTokenRequest {  // @1553552
  string refresh_token = 1;
}
message RefreshTokenResponse {  // @1554959
  string access_token = 2;
  google.protobuf.Timestamp access_token_expires_at = 4;
}
message SelfSignUpRequest {  // @1556982
  string first_name = 1;
  string last_name = 2;
  string email = 3;
  string password = 4;
  bool communication_terms = 5;
  bool communication_system = 6;
  bool communication_product_updates = 7;
  bool communication_promotional = 8;
}
message SelfSignUpResponse {  // @1558796
  assignment.auth.v1.User user = 1;
}
message SendLoginCodeRequest {  // @1560066
  string email = 1;
  bool digit_code = 2;
}
message SendLoginCodeResponse { }  // empty @1561153
message StandardLoginRequest {  // @1562381
  string email = 1;
  string password = 2;
}
message StandardLoginResponse {  // @1564534
  assignment.auth.v1.User user = 1;
  string access_token = 2;
  string refresh_token = 3;
  google.protobuf.Timestamp access_token_expires_at = 4;
  google.protobuf.Timestamp refresh_token_expires_at = 5;
}
```

### assignment.connectivity.v1 -- 4 messages, 0 enums

```proto
message GetConnectivityStatusRequest {  // @1567328
  int64 exam_id = 1;
}
message GetConnectivityStatusResponse {  // @1568565
  assignment.connectivity.v1.Status statuses = 1;  // repeated
}
message SetConnectivityStatusRequest {  // @1569874
  int64 exam_id = 1;
  int64 student_id = 2;
}
message SetConnectivityStatusResponse { }  // empty @1571067
```

### assignment.exam.v1 -- 13 messages, 6 enums

```proto
enum ExamStatus {   EXAM_STATUS_UNSPECIFIED = 0;   EXAM_STATUS_NOT_READY_TO_START = 1;   EXAM_STATUS_READY_TO_START_SOON = 2;   EXAM_STATUS_READY_TO_START = 3;   EXAM_STATUS_STARTED = 4;   EXAM_STATUS_HANDED_IN = 5; }  // assignment.exam.v1.ExamStatus @1573746
enum ToolPermission {   TOOL_PERMISSION_UNSPECIFIED = 0;   TOOL_PERMISSION_MICROPHONE = 1;   TOOL_PERMISSION_CAMERA = 2;   TOOL_PERMISSION_SCREEN_SHARE = 3; }  // assignment.exam.v1.ToolPermission @1574499
enum DurationSetting {   DURATION_SETTING_UNSPECIFIED = 0;   DURATION_SETTING_OFF = 1;   DURATION_SETTING_ON = 2;   DURATION_SETTING_ON_ADJUSTABLE = 3; }  // assignment.exam.v1.DurationSetting @1578653
enum ExamSettingsFlag {   EXAM_SETTINGS_FLAG_UNSPECIFIED = 0;   EXAM_SETTINGS_FLAG_NONE = 1;   EXAM_SETTINGS_FLAG_OFFLINE_START = 2;   EXAM_SETTINGS_FLAG_TRADITIONAL_EXAM = 3;   EXAM_SETTINGS_FLAG_CHEAT_VIRTUAL_MACHINE = 4;   EXAM_SETTINGS_FLAG_LATE_UPLOAD_ONLY = 5;   EXAM_SETTINGS_FLAG_STUDENT_ABSENT = 6;   EXAM_SETTINGS_FLAG_CHEAT_LOST_FOCUS = 7;   EXAM_SETTINGS_FLAG_TEACHER_UPLOAD = 8; }  // assignment.exam.v1.ExamSettingsFlag @1586983
enum ExamState {   EXAM_STATE_UNSPECIFIED = 0;   EXAM_STATE_EMPTY = 1;   EXAM_STATE_DRAFT = 2;   EXAM_STATE_FINAL = 3;   EXAM_STATE_BLANK = 4; }  // assignment.exam.v1.ExamState @1587915
enum ExtendedTimeSetting {   EXTENDED_TIME_SETTING_UNSPECIFIED = 0;   EXTENDED_TIME_SETTING_BY_PERCENT = 1;   EXTENDED_TIME_SETTING_UNLIMITED = 2; }  // assignment.exam.v1.ExtendedTimeSetting @1588568
message ActiveExam {  // @1572671
  int64 exam_id = 1;
  string title = 2;
  int64 student_id = 3;
  string course_title = 4;
}
message ExamDetails {  // @1583824
  int64 student_id = 1;
  int64 exam_id = 2;
  string title = 3;
  string creator_full_name = 4;
  string organization_title = 5;
  string course_title = 6;
  assignment.exam.v1.ExamStatus status = 7;
  google.protobuf.Duration duration = 8;
  google.protobuf.Timestamp start_time = 9;
  google.protobuf.Timestamp student_start_time = 10;
  string description = 11;
  string description_html = 12;
  bool open_book = 13;
  bool remote_proctoring = 14;
  bool right_to_left = 15;
  bool randomize_questions = 16;
  bool randomize_choices = 17;
  bool locked_question_order = 18;
  bool allow_external_paste = 19;
  assignment.exam.v1.ExternalTool external_tools = 20;  // repeated
  assignment.exam.v1.DurationSetting duration_setting = 21;
  int64 org_id = 22;
}
message ExternalTool {  // @1576927
  int64 id = 1;
  int64 organization_id = 2;
  string title = 3;
  string description = 4;
  string logo = 5;
  string landing_page_url = 6;
  bool landing_page_editable = 7;
  string single_page_urls = 8;  // repeated
  string partial_domain_access_urls = 9;  // repeated
  string domain_urls = 10;  // repeated
  assignment.exam.v1.ToolPermission required_permissions = 11;
  bool required_permissions_editable = 12;
}
message GetActiveExamsRequest {  // @1589596
  string email = 1;
}
message GetActiveExamsResponse {  // @1590743
  assignment.exam.v1.ActiveExam active_exams = 1;  // repeated
}
message GetExamDetailsRequest {  // @1592061
  int64 exam_id = 1;
  int64 student_id = 2;
}
message GetExamDetailsResponse {  // @1593297
  assignment.exam.v1.ExamDetails exam_details = 1;
}
message SearchExamDetailsRequest {  // @1597970
  int64 exam_id = 1;
  string email = 2;
}
message SearchExamDetailsResponse {  // @1599238
  assignment.exam.v1.ExamDetails exam_details = 1;
}
message StartOnlineExamRequest {  // @1600466
  int64 student_id = 1;
  int64 exam_id = 2;
}
message StartOnlineExamResponse {  // @1603102
  google.protobuf.Timestamp confirmed_start_time = 1;
  bool spellcheck_allowed = 2;
  bool text_to_speech_allowed = 3;
  bool usb_audio_allowed = 4;
  bool immersive_reader_allowed = 5;
  assignment.exam.v1.ExtendedTimeSetting extended_time_setting = 6;
  int64 extended_time_percent = 7;
}
message StartUSBExamRequest {  // @1605140
  string email = 1;
  int64 exam_id = 2;
  google.protobuf.Timestamp local_start_time = 3;
}
message StartUSBExamResponse {  // @1607959
  google.protobuf.Timestamp confirmed_start_time = 1;
  bool spellcheck_allowed = 2;
  bool text_to_speech_allowed = 3;
  bool usb_audio_allowed = 4;
  bool immersive_reader_allowed = 5;
  assignment.exam.v1.ExtendedTimeSetting extended_time_setting = 6;
  int64 extended_time_percent = 7;
  int64 student_id = 8;
}
```

### assignment.file.v1 -- 4 messages, 0 enums

```proto
message DownloadFileRequest {  // @1609767
  string filename = 1;
}
message DownloadFileResponse {  // @1610866
  bytes content = 1;
}
message UploadFileRequest {  // @1612052
  string filename = 1;
  bytes content = 2;
}
message UploadFileResponse { }  // empty @1613105
```

### assignment.log.v1 -- 4 messages, 1 enums

```proto
enum EntryType {   ENTRY_TYPE_UNSPECIFIED = 0;   ENTRY_TYPE_DEBUG = 1;   ENTRY_TYPE_INFO = 2;   ENTRY_TYPE_WARNING = 3;   ENTRY_TYPE_ERROR = 4;   ENTRY_TYPE_FATAL = 5; }  // assignment.log.v1.EntryType @1613711
message SendLogRequest {  // @1614903
  assignment.log.v1.EntryType entry_type = 1;
  string message = 2;
}
message SendLogResponse { }  // empty @1615901
message StreamLogRequest {  // @1617348
  assignment.log.v1.EntryType entry_type = 1;
  google.protobuf.Timestamp timestamp = 2;
  string target = 3;
  google.protobuf.Struct fields = 4;
  google.protobuf.Struct metadata = 5;
}
message StreamLogResponse { }  // empty @1618548
```

### assignment.proctoring.v1 -- 6 messages, 0 enums

```proto
message CheckProctoringLicensesRequest { }  // empty @1619499
message CheckProctoringLicensesResponse { }  // empty @1620517
message GenerateProctoringTokenRequest {  // @1624316
  string name = 1;
  int64 student_id = 2;
  int64 exam_id = 3;
  string language = 4;
}
message GenerateProctoringTokenResponse {  // @1625827
  string token = 1;
}
message GetDefaultProctoringSettingRequest { }  // empty @1621563
message GetDefaultProctoringSettingResponse {  // @1622743
  int64 value = 1;
}
```

### assignment.question.v1 -- 12 messages, 5 enums

```proto
enum OnlineResourceType {   ONLINE_RESOURCE_TYPE_UNSPECIFIED = 0;   ONLINE_RESOURCE_TYPE_URL = 1;   ONLINE_RESOURCE_TYPE_YOUTUBE = 2;   ONLINE_RESOURCE_TYPE_AUDIO = 3;   ONLINE_RESOURCE_TYPE_PDF = 4; }  // assignment.question.v1.OnlineResourceType @1626667
enum NavigationRestriction {   NAVIGATION_RESTRICTION_UNSPECIFIED = 0;   NAVIGATION_RESTRICTION_SINGLE_PAGE = 1;   NAVIGATION_RESTRICTION_DOMAIN = 2;   NAVIGATION_RESTRICTION_CLOUD_DOCUMENT = 3; }  // assignment.question.v1.NavigationRestriction @1627496
enum QuestionType {   QUESTION_TYPE_UNSPECIFIED = 0;   QUESTION_TYPE_TEXTUAL = 1;   QUESTION_TYPE_SINGLE_CHOICE = 2;   QUESTION_TYPE_MULTIPLE_CHOICE = 3;   QUESTION_TYPE_WORD_GAP = 4; }  // assignment.question.v1.QuestionType @1636806
enum TextualLimit {   TEXTUAL_LIMIT_UNSPECIFIED = 0;   TEXTUAL_LIMIT_NONE = 1;   TEXTUAL_LIMIT_SOFT = 2;   TEXTUAL_LIMIT_HARD = 3; }  // assignment.question.v1.TextualLimit @1640414
enum WordGapMatchingType {   WORD_GAP_MATCHING_TYPE_UNSPECIFIED = 0;   WORD_GAP_MATCHING_TYPE_EXACT = 1;   WORD_GAP_MATCHING_TYPE_CONTAINS = 2; }  // assignment.question.v1.WordGapMatchingType @1643150
message GetQuestionsRequest {  // @1648324
  int64 exam_id = 1;
}
message GetQuestionsResponse {  // @1650557
  assignment.question.v1.OrderItem order = 1;  // repeated
  map<int64, assignment.question.v1.TextualQuestion> textual = 2;
  map<int64, assignment.question.v1.SingleChoiceQuestion> single_choice = 3;
  map<int64, assignment.question.v1.MultipleChoiceQuestion> multiple_choice = 4;
  map<int64, assignment.question.v1.WordGapQuestion> word_gap = 5;
}
message MultipleChoiceQuestion {  // @1635886
  assignment.question.v1.QuestionBase question_base = 1;
  assignment.question.v1.Option options = 2;  // repeated
  int64 max_selections = 3;
}
message OnlineResource {  // @1629386
  assignment.question.v1.OnlineResourceType type = 1;
  string url = 2;
  string title = 3;
  assignment.question.v1.NavigationRestriction navigation_restriction = 4;
}
message Option {  // @1633903
  int64 id = 1;
  string title = 2;
  string about = 3;
  bool right = 4;
}
message OrderItem {  // @1638206
  int64 question_id = 1;
  assignment.question.v1.QuestionType question_type = 2;
}
message QuestionBase {  // @1631856
  int64 question_id = 1;
  string title = 2;
  string about = 3;
  string body_html = 4;
  double max_score = 6;
  assignment.question.v1.OnlineResource online_resources = 10;  // repeated
}
message SingleChoiceQuestion {  // @1639719
  assignment.question.v1.QuestionBase question_base = 1;
  assignment.question.v1.Option options = 2;  // repeated
}
message TextualQuestion {  // @1642266
  assignment.question.v1.QuestionBase question_base = 1;
  assignment.question.v1.TextualLimit limit = 2;
  int32 limit_value = 3;
  string keywords = 4;  // repeated
}
message WordGapBlank {  // @1647075
  string guid = 1;
  assignment.question.v1.WordGapValidation validations = 2;  // repeated
}
message WordGapQuestion {  // @1644313
  assignment.question.v1.QuestionBase question_base = 1;
  assignment.question.v1.WordGapBlank blanks = 2;  // repeated
}
message WordGapValidation {  // @1645712
  assignment.question.v1.WordGapMatchingType matching_type = 1;
  string phrase = 2;
  bool case_sensitive = 3;
}
```

### assignment.service.v1 -- 1 messages, 0 enums

```proto
message EmptyMessage { }  // empty @1672544
```

### assignment.snapshot.v1 -- 2 messages, 1 enums

```proto
enum SystemSnapshotPlatform {   SYSTEM_SNAPSHOT_PLATFORM_UNSPECIFIED = 0;   SYSTEM_SNAPSHOT_PLATFORM_WINDOWS = 1;   SYSTEM_SNAPSHOT_PLATFORM_MACOS = 2; }  // assignment.snapshot.v1.SystemSnapshotPlatform @1654420
message CreateLockdownSystemSnapshotUploadRequest {  // @1656519
  string snapshot_id = 1;
  int64 exam_id = 2;
  int64 student_id = 3;
  string session_id = 4;
  uint32 schema_version = 5;
  uint64 content_size_bytes = 6;
  string crc32c = 7;
  assignment.snapshot.v1.SystemSnapshotPlatform platform = 8;
}
message CreateLockdownSystemSnapshotUploadResponse {  // @1658695
  string snapshot_id = 1;
  string upload_url = 2;
}
```

### assignment.version.v1 -- 2 messages, 0 enums

```proto
message CheckDeprecationRequest {  // @1652343
  int64 exam_id = 1;
  string client_hash = 2;
  string email = 3;
}
message CheckDeprecationResponse {  // @1653687
  bool is_deprecated = 1;
}
```

### directorysync.integration.v1 -- 5 messages, 1 enums

```proto
enum Provider {   PROVIDER_UNSPECIFIED = 0;   PROVIDER_GOOGLE = 1;   PROVIDER_MICROSOFT = 2; }  // directorysync.integration.v1.Provider @1675227
message DisableIntegrationRequest {  // @1678173
  int64 organization_id = 1;
}
message DisableIntegrationResponse {  // @1679390
  int64 task_id = 1;
}
message GetSyncTaskRequest {  // @1680506
  int64 task_id = 1;
}
message GetSyncTaskResponse {  // @1681608
  directorysync.integration.v1.SyncTask task = 1;
}
message SyncTask {  // @1673788
  int64 id = 1;
  directorysync.integration.v1.SyncTask.State state = 2;
  string errors = 3;  // repeated
  google.protobuf.Timestamp updated_at = 4;
}
```

### directorysync.organization.v1 -- 21 messages, 0 enums

```proto
message CreateSyncGroupsRequest {  // @1690162
  int64 organization_id = 1;
  directorysync.organization.v1.SyncGroup groups = 2;  // repeated
}
message CreateSyncGroupsResponse { }  // empty @1691282
message GetIAMUsersForIntegrationRequest {  // @1695152
  int64 organization_id = 1;
  string application = 2;
}
message GetIAMUsersForIntegrationResponse { }  // empty @1696406
message GetOrganizationGroupsRequest {  // @1692424
  int64 organization_id = 1;
}
message GetOrganizationGroupsResponse {  // @1693670
  directorysync.organization.v1.Group groups = 1;  // repeated
}
message GetUsersByGroupsRequest {  // @1697690
  int64 organization_id = 1;
  directorysync.organization.v1.SyncGroup groups = 2;  // repeated
}
message GetUsersByGroupsResponse {  // @1698966
  directorysync.organization.v1.UserToMapRoleTemplate users = 1;  // repeated
}
message GetUsersWithoutRoleTemplatesRequest {  // @1700251
  int64 organization_id = 1;
}
message GetUsersWithoutRoleTemplatesResponse {  // @1701595
  directorysync.organization.v1.UserToMapRoleTemplate users = 1;  // repeated
}
message MapUsersToRoleTemplatesRequest {  // @1705402
  directorysync.organization.v1.UserWithRoleTemplate users = 1;  // repeated
  int64 organization_id = 2;
}
message MapUsersToRoleTemplatesResponse { }  // empty @1706616
message PropagateAllUsersRequest {  // @1707754
  int64 organization_id = 1;
}
message PropagateAllUsersResponse {  // @1708927
  int64 task_id = 1;
}
message SyncAllOrganizationsRequest { }  // empty @1709995
message SyncAllOrganizationsResponse { }  // empty @1710982
message SyncGroup {  // @1684290
  string key = 1;
  int64 template_id = 2;
}
message SyncOrganizationRequest {  // @1712373
  int64 organization_id = 1;
  bool skip_db_sync = 2;
}
message SyncOrganizationResponse {  // @1713635
  int64 task_id = 1;
}
message UserToMapRoleTemplate {  // @1688460
  string email = 1;
  string first_name = 2;
  string last_name = 3;
  string external_id = 4;
  string groups = 5;  // repeated
}
message UserWithRoleTemplate {  // @1703538
  string email = 1;
  string first_name = 2;
  string last_name = 3;
  string external_id = 4;
  int64 template_id = 5;
  string groups = 6;  // repeated
}
```

### event.event.v1 -- 2 messages, 0 enums

```proto
message SendEventRequest {  // @1717702
  google.protobuf.Struct payload = 1;
}
message SendEventResponse { }  // empty @1718600
```

### event.log.v1 -- 10 messages, 2 enums

```proto
enum ExitCode {   EXIT_CODE_UNSPECIFIED = 0;   EXIT_CODE_CLEAN = 1;   EXIT_CODE_CRASH = 2; }  // event.log.v1.ExitCode @1719038
enum ClientType {   CLIENT_TYPE_UNSPECIFIED = 0;   CLIENT_TYPE_WIN = 1;   CLIENT_TYPE_OSX = 2;   CLIENT_TYPE_CHROMEBOOK = 3;   CLIENT_TYPE_IOS = 4;   CLIENT_TYPE_ELECTRON_WINDOWS = 5;   CLIENT_TYPE_ELECTRON_MAC = 6; }  // event.log.v1.ClientType @1724419
message ClientLogEntry {  // @1727427
  string session_id = 1;
  int64 exam_id = 2;
  int64 student_id = 3;
  event.log.v1.ExitCode exit_code = 4;
  event.log.v1.ClientEvent event = 5;
  string message = 6;
  google.protobuf.Timestamp timestamp = 7;
  string os_version = 8;
  string client_version = 9;
  event.log.v1.ClientLogEntry.DisplaySettings display_settings = 10;
  bool touch_enabled = 11;
  bool managed_environment = 12;
  event.log.v1.ClientType client_type = 13;
  string user_agent = 14;
  event.log.v1.ClientLogEntry.Location location = 15;
  int64 index = 16;
}
message ClientLogEntry.DisplaySettings {  // @1730468
  int64 resolution_width = 1;
  int64 resolution_height = 2;
  float pixel_ratio = 3;
  int64 color_depth = 4;
  int64 count = 5;
}
message ClientLogEntry.Location {  // @1732442
  string country = 1;
  string region = 2;
  string city = 3;
  float city_lat = 4;
  float city_long = 5;
}
message ClientLogSession {  // @1734341
  string session_id = 1;
  string student_code = 2;
  string email = 3;
  string client_hash = 4;
  string remote_addr = 5;
}
message SendClientLogRequest {  // @1735826
  event.log.v1.ClientLogEntry entry = 1;
}
message SendClientLogResponse { }  // empty @1736776
message SendClientLogSessionRequest {  // @1737839
  event.log.v1.ClientLogSession session = 1;
}
message SendClientLogSessionResponse { }  // empty @1738877
message StreamClientLogRequest {  // @1739922
  event.log.v1.ClientLogEntry entry = 1;
}
message StreamClientLogResponse { }  // empty @1740896
```

### iam.app.v1 -- 16 messages, 2 enums

```proto
enum InternalApp {   INTERNAL_APP_UNSPECIFIED = 0;   INTERNAL_APP_STUDENT = 1;   INTERNAL_APP_LOCKDOWN = 2;   INTERNAL_APP_LOCKDOWN_PORTAL = 3; }  // iam.app.v1.InternalApp @1742368
enum ExternalApp {   EXTERNAL_APP_UNSPECIFIED = 0;   EXTERNAL_APP_CANVAS = 1; }  // iam.app.v1.ExternalApp @1744907
message ActivateCanvasRequest {  // @1749371
  int64 organization_id = 1;
  string name = 2;
  string description = 3;
  string url = 4;
}
message ActivateCanvasResponse {  // @1751299
  string client_id = 1;
  string client_secret = 2;
  string scope = 3;
  string discovery_url = 4;
}
message DeactivateCanvasRequest {  // @1752794
  int64 organization_id = 1;
}
message DeactivateCanvasResponse { }  // empty @1753819
message GetGrantedAppsByOrgIdRequest {  // @1754968
  int64 organization_id = 1;
}
message GetGrantedAppsByOrgIdResponse {  // @1756235
  iam.app.v1.GrantedApp apps = 1;  // repeated
}
message GetOrgAppsByOrgIdRequest {  // @1757441
  int64 organization_id = 1;
}
message GetOrgAppsByOrgIdResponse {  // @1758655
  iam.app.v1.OrgApp apps = 1;  // repeated
}
message GrantAppToOrgRequest {  // @1760032
  int64 organization_id = 1;
  iam.app.v1.InternalApp app_type = 2;
}
message GrantAppToOrgResponse {  // @1761276
  iam.app.v1.GrantedApp granted_app = 1;
}
message GrantedApp {  // @1744101
  int64 granted_app_id = 1;
  int64 organization_id = 2;
  iam.app.v1.InternalApp app_type = 3;
  google.protobuf.Timestamp created_at = 4;
  google.protobuf.Timestamp updated_at = 5;
}
message OrgApp {  // @1747121
  int64 app_id = 1;
  int64 organization_id = 2;
  iam.app.v1.ExternalApp app_type = 3;
  bool active = 4;
  string url = 5;
  string name = 6;
  string description = 7;
  google.protobuf.Timestamp created_at = 8;
  google.protobuf.Timestamp updated_at = 9;
}
message RevokeAppFromOrgRequest {  // @1762619
  int64 organization_id = 1;
  iam.app.v1.InternalApp app_type = 2;
}
message RevokeAppFromOrgResponse { }  // empty @1763725
message SetOrgAppLinkRequest {  // @1764924
  int64 app_id = 1;
  string url = 2;
}
message SetOrgAppLinkResponse {  // @1766162
  iam.app.v1.OrgApp org_app = 1;
}
```

### iam.idp.v1 -- 21 messages, 3 enums

```proto
enum TransformationType {   TRANSFORMATION_TYPE_UNSPECIFIED = 0;   TRANSFORMATION_TYPE_DIRECT = 1;   TRANSFORMATION_TYPE_LOWERCASE = 2;   TRANSFORMATION_TYPE_UPPERCASE = 3;   TRANSFORMATION_TYPE_REGEX_REPLACE = 4;   TRANSFORMATION_TYPE_CONDITIONAL = 5;   TRANSFORMATION_TYPE_LOOKUP = 6; }  // iam.idp.v1.TransformationType @1767096
enum IdpType {   IDP_TYPE_UNSPECIFIED = 0;   IDP_TYPE_OIDC = 1;   IDP_TYPE_SAML = 2;   IDP_TYPE_LDAP = 3;   IDP_TYPE_GOOGLE = 4;   IDP_TYPE_GITHUB = 5;   IDP_TYPE_MICROSOFT = 6;   IDP_TYPE_CUSTOM_OAUTH2 = 7; }  // iam.idp.v1.IdpType @1773821
enum BindingType {   BINDING_TYPE_UNSPECIFIED = 0;   BINDING_TYPE_HTTP_POST = 1;   BINDING_TYPE_REDIRECT = 2;   BINDING_TYPE_ARTIFACT = 3; }  // iam.idp.v1.BindingType @1777062
message AttributeMapping {  // @1769626
  int64 id = 1;
  int64 identity_provider_id = 2;
  string source_attribute = 3;
  string target_attribute = 4;
  iam.idp.v1.TransformationType transformation_type = 5;
  string transformation_rule = 6;
  string default_value = 7;
  bool required = 8;
}
message AttributeMappingCreateInput {  // @1772487
  string source_attribute = 1;
  string target_attribute = 2;
  iam.idp.v1.TransformationType transformation_type = 3;
  string transformation_rule = 4;
  string default_value = 5;
  bool required = 6;
}
message DeleteIdpRequest {  // @1783809
  int64 id = 1;
}
message DeleteIdpResponse { }  // empty @1784737
message DisableIdpRequest {  // @1785722
  int64 id = 1;
}
message DisableIdpResponse { }  // empty @1786662
message EnableIdpRequest {  // @1787642
  int64 id = 1;
}
message EnableIdpResponse { }  // empty @1788570
message GetIdpByIdRequest {  // @1792077
  int64 id = 1;
}
message GetIdpByIdResponse {  // @1793202
  iam.idp.v1.IdentityProvider identity_provider = 1;
}
message GetIdpsWithFilterRequest {  // @1789708
  int64 organization_id = 1;
  bool enabled = 2;
}
message GetIdpsWithFilterResponse {  // @1790965
  iam.idp.v1.IdentityProvider identity_providers = 1;  // repeated
}
message IdentityProvider {  // @1781721
  int64 id = 1;
  int64 organization_id = 2;
  string name = 3;
  iam.idp.v1.IdpType type = 4;
  bool enabled = 5;
  map<string, string> attributes = 6;
  iam.idp.v1.AttributeMapping attribute_mappings = 7;  // repeated
  google.protobuf.Timestamp created_at = 9;
  google.protobuf.Timestamp updated_at = 10;
  iam.idp.v1.OidcConfig oidc = 11;
  iam.idp.v1.OAuth2Config oauth2 = 12;
  iam.idp.v1.SamlConfig saml = 13;
  iam.idp.v1.LdapConfig ldap = 14;
}
message LdapConfig { }  // empty @1779171
message LinkIdpToUserRequest {  // @1795065
  int64 idp_id = 1;
  int64 user_id = 2;
  int64 organization_id = 3;
  string external_id = 4;
  string external_username = 5;
}
message LinkIdpToUserResponse { }  // empty @1796457
message OAuth2Config { }  // empty @1776542
message OidcConfig {  // @1775402
  string issuer = 1;
  string client_id = 2;
  string client_secret = 3;
  string scopes = 4;  // repeated
}
message RegisterIdpRequest {  // @1798483
  int64 organization_id = 1;
  string name = 2;
  iam.idp.v1.IdpType type = 3;
  iam.idp.v1.OidcConfig oidc = 4;
  iam.idp.v1.OAuth2Config oauth2 = 5;
  iam.idp.v1.SamlConfig saml = 6;
  iam.idp.v1.LdapConfig ldap = 7;
  map<string, string> attributes = 8;
  iam.idp.v1.AttributeMappingCreateInput attribute_mappings = 9;  // repeated
}
message RegisterIdpResponse {  // @1800311
  iam.idp.v1.IdentityProvider idp = 1;
}
message SamlConfig {  // @1778232
  string metadata_url = 1;
  iam.idp.v1.BindingType binding = 2;
}
```

### iam.link.v1 -- 8 messages, 0 enums

```proto
message CollectTokensRequest {  // @1801347
  string code = 1;
}
message CollectTokensResponse {  // @1802946
  string access_token = 1;
  string refresh_token = 2;
  google.protobuf.Timestamp access_expires_at = 3;
  google.protobuf.Timestamp refresh_expires_at = 4;
  string id_token = 5;
}
message CreateLoginLinkRequest {  // @1804677
  string user_email = 1;
  iam.app.v1.InternalApp app = 2;
}
message CreateLoginLinkResponse {  // @1806096
  string login_link = 1;
  google.protobuf.Timestamp expires_at = 2;
}
message CreateLogoutLinkRequest {  // @1807551
  iam.app.v1.InternalApp app = 1;
  string id_token = 2;
}
message CreateLogoutLinkResponse {  // @1808814
  string logout_link = 1;
}
message CreateZitadelAuthLoginLinkRequest { }  // empty @1809913
message CreateZitadelAuthLoginLinkResponse {  // @1811118
  string login_link = 1;
}
```

### iam.organization.v1 -- 29 messages, 0 enums

```proto
message ActivateOrgRequest {  // @1818996
  int64 organization_id = 1;
}
message ActivateOrgResponse {  // @1820150
  iam.organization.v1.Organization organization = 1;
}
message AddFeatureToOrgsRequest {  // @1821466
  int64 feature_id = 1;
  int64 organization_ids = 2;  // repeated
}
message AddFeatureToOrgsResponse { }  // empty @1822572
message CreateFeatureRequest {  // @1823764
  string name = 1;
  string description = 2;
}
message CreateFeatureResponse {  // @1825012
  iam.organization.v1.Feature feature = 1;
}
message CreateOrgRequest {  // @1826567
  string name = 1;
  string display_name = 2;
  string time_zone = 3;
  string default_language = 4;
}
message CreateOrgResponse {  // @1827995
  iam.organization.v1.Organization organization = 1;
}
message DeactivateOrgRequest {  // @1829122
  int64 organization_id = 1;
}
message DeactivateOrgResponse {  // @1830302
  iam.organization.v1.Organization organization = 1;
}
message DeleteFeatureRequest {  // @1831416
  int64 feature_id = 1;
}
message DeleteFeatureResponse { }  // empty @1832409
message DeleteOrgRequest {  // @1833447
  int64 organization_id = 1;
}
message DeleteOrgResponse { }  // empty @1834397
message Feature {  // @1812673
  int64 feature_id = 1;
  string name = 2;
  string description = 3;
  google.protobuf.Timestamp created_at = 4;
}
message GetAllOrgsWithFeatureRequest {  // @1835510
  int64 feature_id = 1;
}
message GetAllOrgsWithFeatureResponse {  // @1836730
  int64 organization_ids = 1;  // repeated
}
message GetOrgByIDRequest {  // @1840471
  int64 id = 1;
}
message GetOrgByIDResponse {  // @1841558
  iam.organization.v1.Organization organization = 1;
}
message GetOrgFeaturesRequest {  // @1842703
  int64 organization_id = 1;
}
message GetOrgFeaturesResponse {  // @1843869
  iam.organization.v1.OrgFeatureWithTime org_features = 1;  // repeated
}
message GetOrgsWithFilterRequest {  // @1838129
  bool active = 1;
  string name = 2;
}
message GetOrgsWithFilterResponse {  // @1839345
  iam.organization.v1.Organization organizations = 1;  // repeated
}
message Organization {  // @1817128
  int64 organization_id = 1;
  string name = 2;
  string display_name = 3;
  string time_zone = 4;
  string default_language = 5;
  bool active = 6;
  google.protobuf.Timestamp created_at = 7;
  google.protobuf.Timestamp updated_at = 8;
  string identity_platform_id = 9;
}
message OrgFeatureWithTime {  // @1814364
  int64 organization_id = 1;
  iam.organization.v1.Feature feature = 2;
  google.protobuf.Timestamp created_at = 3;
}
message RemoveFeatureFromOrgsRequest {  // @1845297
  int64 feature_id = 1;
  int64 organization_ids = 2;  // repeated
}
message RemoveFeatureFromOrgsResponse { }  // empty @1846463
message UpdateOrgRequest {  // @1848253
  int64 organization_id = 1;
  string name = 2;
  string display_name = 3;
  string time_zone = 4;
  string default_language = 5;
}
message UpdateOrgResponse {  // @1849606
  iam.organization.v1.Organization organization = 1;
}
```

### iam.permission.v1 -- 41 messages, 0 enums

```proto
message CreateCustomTemplateRequest {  // @1857046
  int64 organization_id = 1;
  string name = 2;
  string description = 3;
  string permission_keys = 4;  // repeated
}
message CreateCustomTemplateResponse {  // @1858566
  iam.permission.v1.Template template = 1;
}
message CreateDigiexamInternalTemplateMappingRequest {  // @1860061
  int64 template_id = 1;
  int64 user_id = 2;
}
message CreateDigiexamInternalTemplateMappingResponse {  // @1861605
  iam.permission.v1.TemplateMapping mapping = 1;
}
message CreateDigiexamInternalTemplateRequest {  // @1863280
  string name = 1;
  string description = 2;
  string permission_keys = 3;  // repeated
}
message CreateDigiexamInternalTemplateResponse {  // @1864818
  iam.permission.v1.Template template = 1;
}
message CreateGlobalTemplateRequest {  // @1866370
  string name = 1;
  string description = 2;
  string permission_keys = 3;  // repeated
}
message CreateGlobalTemplateResponse {  // @1867781
  iam.permission.v1.Template template = 1;
}
message CreateTemplateMappingRequest {  // @1869320
  int64 organization_id = 1;
  int64 template_id = 2;
  int64 user_id = 3;
}
message CreateTemplateMappingResponse {  // @1870777
  iam.permission.v1.TemplateMapping mapping = 1;
}
message DeleteCustomTemplateRequest {  // @1871968
  int64 id = 1;
}
message DeleteCustomTemplateResponse { }  // empty @1873035
message DeleteGlobalTemplateRequest {  // @1874140
  int64 id = 1;
}
message DeleteGlobalTemplateResponse { }  // empty @1875207
message GetAllDigiexamInternalPermissionsRequest { }  // empty @1876282
message GetAllDigiexamInternalPermissionsResponse {  // @1877596
  iam.permission.v1.Permission permissions = 1;  // repeated
}
message GetAllPermissionsRequest { }  // empty @1878677
message GetAllPermissionsResponse {  // @1879804
  iam.permission.v1.Permission permissions = 1;  // repeated
}
message GetCustomTemplatesByOrgIDRequest {  // @1881068
  int64 organization_id = 1;
}
message GetCustomTemplatesByOrgIDResponse {  // @1882385
  iam.permission.v1.Template templates = 1;  // repeated
}
message GetGlobalTemplatesRequest { }  // empty @1883438
message GetGlobalTemplatesResponse {  // @1884559
  iam.permission.v1.Template templates = 1;  // repeated
}
message GetTemplateMappingsByUserIDsRequest {  // @1886003
  int64 organization_id = 1;
  int64 user_ids = 2;  // repeated
}
message GetTemplateMappingsByUserIDsResponse {  // @1887437
  iam.permission.v1.TemplateMapping mappings = 1;  // repeated
}
message GetUserIDsByPermissionRequest {  // @1889098
  int64 organization_id = 1;
  string permission = 2;
  bool active = 3;
}
message GetUserIDsByPermissionResponse {  // @1890567
  int64 user_ids = 1;  // repeated
}
message Permission {  // @1850802
  string key = 1;
  string description = 2;
}
message Template {  // @1852886
  int64 id = 1;
  int64 organization_id = 2;
  string name = 3;
  string description = 4;
  string permission_keys = 5;  // repeated
  google.protobuf.Timestamp created_at = 6;
  google.protobuf.Timestamp updated_at = 7;
}
message TemplateMapping {  // @1855078
  int64 mapping_id = 1;
  int64 user_id = 2;
  iam.permission.v1.Template template = 3;
  google.protobuf.Timestamp created_at = 4;
  google.protobuf.Timestamp updated_at = 5;
}
message UpdateCustomTemplateRequest {  // @1892478
  int64 id = 1;
  string name = 2;
  string description = 3;
  string permissions_to_add = 4;  // repeated
  string permissions_to_remove = 5;  // repeated
}
message UpdateCustomTemplateResponse {  // @1893996
  iam.permission.v1.Template template = 1;
}
message UpdateDigiexamInternalTemplateMappingRequest {  // @1895502
  int64 mapping_id = 1;
  int64 template_id = 2;
}
message UpdateDigiexamInternalTemplateMappingResponse {  // @1897049
  iam.permission.v1.TemplateMapping mapping = 1;
}
message UpdateDigiexamInternalTemplateRequest {  // @1899098
  int64 id = 1;
  string name = 2;
  string description = 3;
  string permissions_to_add = 4;  // repeated
  string permissions_to_remove = 5;  // repeated
}
message UpdateDigiexamInternalTemplateResponse {  // @1900736
  iam.permission.v1.Template template = 1;
}
message UpdateGlobalTemplateRequest {  // @1902624
  int64 id = 1;
  string name = 2;
  string description = 3;
  string permissions_to_add = 4;  // repeated
  string permissions_to_remove = 5;  // repeated
}
message UpdateGlobalTemplateResponse {  // @1904103
  iam.permission.v1.Template template = 1;
}
message UpdateTemplateMappingByEmailRequest {  // @1905700
  string email = 1;
  int64 organization_id = 2;
  int64 template_id = 3;
}
message UpdateTemplateMappingByEmailResponse {  // @1907240
  iam.permission.v1.TemplateMapping mapping = 1;
}
message UpdateTemplateMappingRequest {  // @1908854
  int64 mapping_id = 1;
  int64 organization_id = 2;
  int64 template_id = 3;
}
message UpdateTemplateMappingResponse {  // @1910314
  iam.permission.v1.TemplateMapping mapping = 1;
}
```

### iam.policy.v1 -- 27 messages, 1 enums

```proto
enum MfaEnforcement {   MFA_ENFORCEMENT_UNSPECIFIED = 0;   MFA_ENFORCEMENT_NONE = 1;   MFA_ENFORCEMENT_FORCE_ALL = 2;   MFA_ENFORCEMENT_FORCE_LOCAL_ONLY = 3; }  // iam.policy.v1.MfaEnforcement @1914971
message GetDefaultPolicySetRequest { }  // empty @1924321
message GetDefaultPolicySetResponse {  // @1925404
  iam.policy.v1.PolicySet policy_set = 1;
}
message GetPolicySetByOrgIdRequest {  // @1926629
  int64 organization_id = 1;
}
message GetPolicySetByOrgIdResponse {  // @1927857
  iam.policy.v1.PolicySet policy_set = 1;
}
message LockoutPolicy {  // @1911631
  int32 max_password_attempts = 1;
  int32 max_otp_attempts = 2;
}
message LoginLifetimePolicy {  // @1913764
  int32 password_check_lifetime_hours = 1;
  int32 external_login_check_lifetime_hours = 2;
  int32 mfa_init_skip_lifetime_hours = 3;
  int32 second_factor_check_lifetime_hours = 4;
  int32 multi_factor_check_lifetime_hours = 5;
}
message MfaPolicy {  // @1916867
  iam.policy.v1.MfaEnforcement enforcement_mode = 1;
  bool passwordless_allowed = 2;
  bool allow_totp = 3;
  bool allow_passkey = 4;
  bool allow_sms = 5;
  bool allow_email = 6;
}
message PasswordPolicy {  // @1919573
  int32 min_length = 1;
  bool require_uppercase = 2;
  bool require_lowercase = 3;
  bool require_number = 4;
  bool require_symbol = 5;
  int32 max_age_days = 6;
  int32 expire_warning_days = 7;
}
message PolicySet {  // @1922742
  int64 policy_set_id = 1;
  int64 organization_id = 2;
  bool is_default = 3;
  google.protobuf.Timestamp created_at = 4;
  google.protobuf.Timestamp updated_at = 5;
  iam.policy.v1.PasswordPolicy password = 6;
  iam.policy.v1.MfaPolicy mfa = 7;
  iam.policy.v1.LockoutPolicy lockout = 8;
  iam.policy.v1.LoginLifetimePolicy lifetime = 9;
}
message ResetOrgPoliciesToDefaultRequest {  // @1929116
  int64 organization_id = 1;
}
message ResetOrgPoliciesToDefaultResponse {  // @1930425
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdateDefaultLockoutPolicyRequest {  // @1931806
  int32 max_password_attempts = 1;
  int32 max_otp_attempts = 2;
}
message UpdateDefaultLockoutPolicyResponse {  // @1933190
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdateDefaultLoginLifetimePolicyRequest {  // @1935115
  int32 password_check_lifetime_hours = 1;
  int32 external_login_check_lifetime_hours = 2;
  int32 mfa_init_skip_lifetime_hours = 3;
  int32 second_factor_check_lifetime_hours = 4;
  int32 multi_factor_check_lifetime_hours = 5;
}
message UpdateDefaultLoginLifetimePolicyResponse {  // @1936963
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdateDefaultMfaPolicyRequest {  // @1939138
  iam.policy.v1.MfaEnforcement enforcement_mode = 1;
  bool passwordless_allowed = 2;
  bool allow_totp = 3;
  bool allow_passkey = 4;
  bool allow_sms = 5;
  bool allow_email = 6;
}
message UpdateDefaultMfaPolicyResponse {  // @1940857
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdateDefaultPasswordPolicyRequest {  // @1943162
  int32 min_length = 1;
  bool require_uppercase = 2;
  bool require_lowercase = 3;
  bool require_number = 4;
  bool require_symbol = 5;
  int32 max_age_days = 6;
  int32 expire_warning_days = 7;
}
message UpdateDefaultPasswordPolicyResponse {  // @1945070
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdateLockoutPolicyRequest {  // @1946698
  int64 organization_id = 1;
  int32 max_password_attempts = 2;
  int32 max_otp_attempts = 3;
}
message UpdateLockoutPolicyResponse {  // @1948103
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdateLoginLifetimePolicyRequest {  // @1950363
  int64 organization_id = 1;
  int32 password_check_lifetime_hours = 2;
  int32 external_login_check_lifetime_hours = 3;
  int32 mfa_init_skip_lifetime_hours = 4;
  int32 second_factor_check_lifetime_hours = 5;
  int32 multi_factor_check_lifetime_hours = 6;
}
message UpdateLoginLifetimePolicyResponse {  // @1952232
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdateMfaPolicyRequest {  // @1954524
  int64 organization_id = 1;
  iam.policy.v1.MfaEnforcement enforcement_mode = 2;
  bool passwordless_allowed = 3;
  bool allow_totp = 4;
  bool allow_passkey = 5;
  bool allow_sms = 6;
  bool allow_email = 7;
}
message UpdateMfaPolicyResponse {  // @1956264
  iam.policy.v1.PolicySet policy_set = 1;
}
message UpdatePasswordPolicyRequest {  // @1958729
  int64 organization_id = 1;
  int32 min_length = 2;
  bool require_uppercase = 3;
  bool require_lowercase = 4;
  bool require_number = 5;
  bool require_symbol = 6;
  int32 max_age_days = 7;
  int32 expire_warning_days = 8;
}
message UpdatePasswordPolicyResponse {  // @1960658
  iam.policy.v1.PolicySet policy_set = 1;
}
```

### iam.user.v1 -- 17 messages, 0 enums

```proto
message ActivateUserRequest {  // @1985549
  int64 user_id = 1;
}
message ActivateUserResponse { }  // empty @1986519
message CreateUsersRequest {  // @1970006
  int64 organization_id = 1;
  iam.user.v1.UserInput users = 2;  // repeated
}
message CreateUsersResponse {  // @1971204
  iam.user.v1.User users = 1;  // repeated
}
message DeactivateUserRequest {  // @1987573
  int64 user_id = 1;
}
message DeactivateUserResponse { }  // empty @1988567
message DeleteUserRequest {  // @1989589
  int64 user_id = 1;
}
message DeleteUserResponse { }  // empty @1990535
message GetUserByIdRequest {  // @1972313
  int64 user_id = 1;
}
message GetUserByIdResponse {  // @1973414
  iam.user.v1.User user = 1;
}
message GetUsersByEmailsRequest {  // @1980275
  int64 organization_id = 1;
  string emails = 2;  // repeated
  bool active = 3;
}
message GetUsersByEmailsResponse {  // @1981621
  iam.user.v1.User users = 1;  // repeated
}
message GetUsersByIdsRequest {  // @1974725
  int64 user_ids = 1;  // repeated
  int64 organization_id = 2;
}
message GetUsersByIdsResponse {  // @1975934
  iam.user.v1.User users = 1;  // repeated
}
message GetUsersWithFilterRequest {  // @1977472
  int64 organization_id = 1;
  bool active = 2;
  string email = 3;
}
message GetUsersWithFilterResponse {  // @1978769
  iam.user.v1.User users = 1;  // repeated
}
message UserInput {  // @1968357
  string email = 1;
  string first_name = 2;
  string last_name = 3;
  string time_zone = 4;
  google.protobuf.Struct attributes = 5;
}
```

### intserver.ping.v1 -- 2 messages, 0 enums

```proto
message PingRequest { }  // empty @2002448
message PingResponse { }  // empty @2003231
```

### lis.integration.v1 -- 27 messages, 3 enums

```proto
enum AuthenticationProviderType {   AUTHENTICATION_PROVIDER_TYPE_UNSPECIFIED = 0;   AUTHENTICATION_PROVIDER_TYPE_APPLE = 1;   AUTHENTICATION_PROVIDER_TYPE_CANVAS = 2;   AUTHENTICATION_PROVIDER_TYPE_CAS = 3;   AUTHENTICATION_PROVIDER_TYPE_CLEVER = 4;   AUTHENTICATION_PROVIDER_TYPE_FACEBOOK = 5;   AUTHENTICATION_PROVIDER_TYPE_GITHUB = 6;   AUTHENTICATION_PROVIDER_TYPE_GOOGLE = 7;   AUTHENTICATION_PROVIDER_TYPE_LDAP = 8;   AUTHENTICATION_PROVIDER_TYPE_LINKEDIN = 9;   AUTHENTICATION_PROVIDER_TYPE_MICROSOFT = 10;   AUTHENTICATION_PROVIDER_TYPE_OPENID_CONNECT = 11;   AUTHENTICATION_PROVIDER_TYPE_SAML = 12; }  // lis.integration.v1.AuthenticationProviderType @2004878
enum ProviderType {   PROVIDER_TYPE_UNSPECIFIED = 0;   PROVIDER_TYPE_CANVAS = 1; }  // lis.integration.v1.ProviderType @2008422
enum ConnectionStatus {   CONNECTION_STATUS_UNSPECIFIED = 0;   CONNECTION_STATUS_INCOMPLETE = 1;   CONNECTION_STATUS_CONNECTED = 2;   CONNECTION_STATUS_REAUTHORIZATION_REQUIRED = 3;   CONNECTION_STATUS_DISCONNECTED = 4; }  // lis.integration.v1.ConnectionStatus @2009119
message AuthenticationProviderInfo {  // @2007583
  string authentication_provider_id = 1;
  lis.integration.v1.AuthenticationProviderType type = 2;
  int64 position = 3;
  bool fallback = 4;
}
message ConnectionInfo {  // @2011408
  int64 provider_instance_id = 1;
  lis.integration.v1.ProviderType provider_type = 2;
  string base_url = 3;
  bool connected = 4;
  google.protobuf.Timestamp created_at = 5;
  string auth_url = 6;
  lis.integration.v1.ConnectionStatus status = 7;
}
message CreateConnectionRequest {  // @2015253
  int64 org_id = 1;
  lis.integration.v1.ProviderType provider_type = 2;
  string base_url = 3;
  string client_id = 4;
  string client_secret = 5;
}
message CreateConnectionResponse {  // @2017421
  int64 connection_id = 1;
  string authorize_url = 2;
}
message DeleteConnectionRequest {  // @2018951
  int64 org_id = 1;
  int64 connection_id = 2;
}
message DeleteConnectionResponse { }  // empty @2020078
message GetProviderRolesRequest {  // @2021217
  int64 provider_instance_id = 1;
}
message GetProviderRolesResponse {  // @2022383
  lis.integration.v1.Role roles = 1;  // repeated
}
message GetProviderUserIDRequest {  // @2023997
  int64 org_id = 1;
  string email = 2;
  lis.integration.v1.ProviderType provider_type = 3;
}
message GetProviderUserIDResponse {  // @2025395
  int64 provider_user_id = 1;
}
message ListAuthenticationProvidersRequest {  // @2026983
  int64 provider_instance_id = 1;
  lis.integration.v1.ProviderType provider_type = 2;
}
message ListAuthenticationProvidersResponse {  // @2028570
  lis.integration.v1.AuthenticationProviderInfo providers = 1;  // repeated
}
message ListConnectionsRequest {  // @2029789
  int64 org_id = 1;
}
message ListConnectionsResponse {  // @2031006
  lis.integration.v1.ConnectionInfo connections = 1;  // repeated
}
message RegisterDigiexamIAMRequest {  // @2032407
  int64 org_id = 1;
  lis.integration.v1.ProviderType provider_type = 2;
}
message RegisterDigiexamIAMResponse { }  // empty @2033554
message Role {  // @2013011
  string id = 1;
  string name = 2;
}
message SetFallbackAuthenticationProviderRequest {  // @2038193
  int64 provider_instance_id = 1;
  lis.integration.v1.ProviderType provider_type = 2;
  string fallback_authentication_provider_id = 3;
}
message SetFallbackAuthenticationProviderResponse {  // @2039888
  lis.integration.v1.AuthenticationProviderInfo fallback_provider = 1;
}
message SetupAuthenticationProviderRequest {  // @2035040
  int64 provider_instance_id = 1;
  lis.integration.v1.AuthenticationProviderType type = 2;
}
message SetupAuthenticationProviderResponse { }  // empty @2036302
message StartConnectionReauthorizationRequest {  // @2041449
  int64 org_id = 1;
  int64 provider_instance_id = 2;
}
message StartConnectionReauthorizationResponse {  // @2042910
  string authorize_url = 1;
}
message SyncProviderUsersRequest {  // @2044422
  int64 org_id = 1;
  lis.integration.v1.ProviderType provider_type = 3;
}
message SyncProviderUsersResponse { }  // empty @2045545
message UnregisterDigiexamIAMRequest {  // @2046720
  int64 org_id = 1;
}
message UnregisterDigiexamIAMResponse { }  // empty @2047804
```

### lis.monitoring.v1 -- 8 messages, 1 enums

```proto
enum SessionActivityType {   SESSION_ACTIVITY_TYPE_UNSPECIFIED = 0;   SESSION_ACTIVITY_TYPE_IN_PROGRESS = 1;   SESSION_ACTIVITY_TYPE_FINISHED = 2; }  // lis.monitoring.v1.SessionActivityType @2053686
message Filter {  // @2055041
  string user_ids = 1;  // repeated
  int64 learning_resource_id = 2;
  bool only_active = 3;
}
message GetSessionActivitiesRequest {  // @2056406
  lis.monitoring.v1.Filter filter = 1;
}
message GetSessionActivitiesResponse {  // @2057666
  lis.monitoring.v1.LearningResourceUserSessionActivities learning_resource_user_session_activities = 1;  // repeated
}
message LearningResourceUserSessionActivities {  // @2052811
  int64 learning_resource_id = 1;
  lis.monitoring.v1.UserSessionActivities user_session_activities = 2;  // repeated
}
message SendSessionActivityRequest {  // @2059796
  string session_id = 1;
  string user_id = 2;
  int64 learning_resource_id = 3;
  lis.monitoring.v1.SessionActivityType type = 4;
  google.protobuf.Timestamp ping_called_at = 5;
}
message SendSessionActivityResponse { }  // empty @2061221
message SessionActivity {  // @2049662
  string session_id = 1;
  bool is_active = 2;
  google.protobuf.Timestamp created_at = 3;
  google.protobuf.Timestamp accessed_at = 4;
}
message UserSessionActivities {  // @2051198
  string user_id = 1;
  lis.monitoring.v1.SessionActivity session_activities = 2;  // repeated
}
```

### lis.resource.v1 -- 80 messages, 4 enums

```proto
enum ExternalResourceType {   EXTERNAL_RESOURCE_TYPE_UNSPECIFIED = 0;   EXTERNAL_RESOURCE_TYPE_ASSIGNMENT = 1;   EXTERNAL_RESOURCE_TYPE_CLASSIC_QUIZ = 2;   EXTERNAL_RESOURCE_TYPE_NEW_QUIZ = 3;   EXTERNAL_RESOURCE_TYPE_DISCUSSION_TOPIC = 4;   EXTERNAL_RESOURCE_TYPE_LTI_TOOL = 5; }  // lis.resource.v1.ExternalResourceType @2063397
enum ActivityType {   ACTIVITY_TYPE_UNSPECIFIED = 0;   ACTIVITY_TYPE_ASSIGNMENT = 1;   ACTIVITY_TYPE_QUIZ = 2;   ACTIVITY_TYPE_DISCUSSION = 3;   ACTIVITY_TYPE_TOOL = 4; }  // lis.resource.v1.ActivityType @2074850
enum UserSchedulingStatus {   USER_SCHEDULING_STATUS_UNSPECIFIED = 0;   USER_SCHEDULING_STATUS_PENDING = 1;   USER_SCHEDULING_STATUS_SCHEDULED = 2;   USER_SCHEDULING_STATUS_FAILED = 3;   USER_SCHEDULING_STATUS_OFF_SCHEDULE = 4;   USER_SCHEDULING_STATUS_DONE = 5;   USER_SCHEDULING_STATUS_CANCELLED = 6; }  // lis.resource.v1.UserSchedulingStatus @2085559
enum SetupSchedulingStatus {   SETUP_SCHEDULING_STATUS_UNSPECIFIED = 0;   SETUP_SCHEDULING_STATUS_PENDING = 1;   SETUP_SCHEDULING_STATUS_SCHEDULED = 2;   SETUP_SCHEDULING_STATUS_FAILED = 3;   SETUP_SCHEDULING_STATUS_DONE = 4;   SETUP_SCHEDULING_STATUS_CANCELLED = 5; }  // lis.resource.v1.SetupSchedulingStatus @2086642
message AccessUser {  // @2062375
  int64 iam_user_id = 1;
  string provider_user_id = 2;
}
message Activity {  // @2104770
  string id = 1;
  lis.resource.v1.ExternalResourceType type = 2;
  string title = 3;
  string url = 4;
  string description_html = 5;
  bool published = 6;
  google.protobuf.Timestamp due_at = 7;
  google.protobuf.Timestamp lock_at = 8;
  google.protobuf.Timestamp unlock_at = 9;
  map<string, int64> source_refs = 10;
}
message ActivityCounts {  // @2101877
  int32 assignment = 1;
  int32 discussion = 2;
  int32 quiz_classic = 3;
  int32 quiz_new = 4;
}
message ActivityItem {  // @2066684
  string id = 1;
  string course_id = 2;
  string title = 3;
  map<string, string> source_refs = 4;
  google.protobuf.Timestamp due_at = 5;
  google.protobuf.Timestamp available_from = 6;
  google.protobuf.Timestamp available_until = 7;
  int64 learning_resource_id = 8;
  bool lockdown = 9;
  lis.resource.v1.ExternalResourceType external_type = 10;
  string url = 11;
  string description_html = 12;
}
message ActivityRef {  // @2069350
  int64 provider_instance_id = 1;
  string external_course_id = 2;
  string external_resource_id = 3;
  lis.resource.v1.ExternalResourceType external_type = 4;
}
message CancelScheduledAccessSetupRequest {  // @2106934
  int64 id = 1;
  int64 provider_instance_id = 2;
}
message CancelScheduledAccessSetupResponse { }  // empty @2108181
message Course {  // @2071266
  int64 id = 1;
  string name = 2;
  string code = 3;
  string url = 4;
  lis.resource.v1.CourseTerm term = 5;
}
message CourseRef {  // @2074054
  int64 provider_instance_id = 1;
  string external_course_id = 2;
}
message CourseTerm {  // @2072755
  int64 id = 1;
  string name = 2;
}
message CourseWithActivities {  // @2099839
  int64 id = 1;
  string name = 2;
  string code = 3;
  string url = 4;
  lis.resource.v1.CourseTerm term = 5;
  lis.resource.v1.ActivityCounts counts = 6;
  lis.resource.v1.Activity activities = 7;  // repeated
}
message CreateScheduledAccessSetupRequest {  // @2110266
  lis.resource.v1.ActivityRef activity = 1;
  google.protobuf.Timestamp available_from = 2;
  google.protobuf.Timestamp available_until = 3;
  google.protobuf.Timestamp due_at = 4;
  lis.resource.v1.AccessUser users = 5;  // repeated
  string name = 6;
}
message CreateScheduledAccessSetupResponse {  // @2111960
  lis.resource.v1.ScheduledAccessSetup scheduled_access_setup = 1;
}
message CurrentLISControlledActivity {  // @2134293
  lis.resource.v1.ActivityRef activity = 1;
  int64 learning_resource_id = 2;
  string title = 3;
}
message DeletePolicyRequest {  // @2113118
  int64 learning_resource_id = 1;
}
message DeletePolicyResponse { }  // empty @2114105
message DeleteScheduledAccessSetupRequest {  // @2115439
  int64 id = 1;
  int64 provider_instance_id = 2;
}
message DeleteScheduledAccessSetupResponse { }  // empty @2116686
message EnsureResourceRequest {  // @2118609
  int64 org_id = 1;
  int64 provider_instance_id = 2;
  string external_course_id = 3;
  string external_resource_id = 4;
  lis.resource.v1.ExternalResourceType external_type = 5;
  string external_url = 6;
}
message EnsureResourceResponse {  // @2120398
  int64 learning_resource_id = 1;
  lis.resource.v1.FullResourceView view = 2;
}
message ExtendScheduledAccessUserTimeRequest {  // @2122105
  int64 provider_instance_id = 1;
  int64 id = 2;
  google.protobuf.Duration time_extension = 3;
}
message ExtendScheduledAccessUserTimeResponse { }  // empty @2123455
message FullResourceView {  // @2082703
  lis.resource.v1.LearningResource resource = 1;
  lis.resource.v1.ResourceBinding bindings = 2;  // repeated
  lis.resource.v1.ResourcePolicy policy = 3;
}
message GetActivitiesWithFilterRequest {  // @2125001
  int64 provider_instance_id = 1;
  int64 provider_user_id = 2;
  string course_id = 3;
}
message GetActivitiesWithFilterResponse {  // @2126469
  lis.resource.v1.ActivityItem activities = 1;  // repeated
}
message GetCoursesByUserIDRequest {  // @2127899
  int64 provider_instance_id = 1;
  int64 provider_user_id = 2;
}
message GetCoursesByUserIDResponse {  // @2129195
  lis.resource.v1.Course courses = 1;  // repeated
}
message GetCurrentLISControlledActivitiesByCourseRequest {  // @2135851
  lis.resource.v1.CourseRef course = 1;
}
message GetCurrentLISControlledActivitiesByCourseResponse {  // @2137343
  lis.resource.v1.CurrentLISControlledActivity activities = 1;  // repeated
}
message GetGroupsByCourseRequest {  // @2130369
  lis.resource.v1.CourseRef course = 1;
}
message GetGroupsByCourseResponse {  // @2131539
  lis.resource.v1.Group groups = 1;  // repeated
}
message GetResourceLockForUsersRequest {  // @2138777
  int64 learning_resource_id = 1;
  string user_ids = 2;  // repeated
}
message GetResourceLockForUsersResponse {  // @2140160
  lis.resource.v1.GetResourceLockForUsersResponse.UserResourceLock user_resource_locks = 1;  // repeated
}
message GetResourceLockForUsersResponse.UserResourceLock {  // @2141647
  string user_id = 1;
  bool locked = 2;
}
message GetResourceRequest {  // @2142959
  int64 learning_resource_id = 1;
}
message GetResourceResponse {  // @2144056
  lis.resource.v1.FullResourceView view = 1;
}
message GetScheduledAccessSetupsByActivityRequest {  // @2145351
  lis.resource.v1.ActivityRef activity = 1;
}
message GetScheduledAccessSetupsByActivityResponse {  // @2146779
  lis.resource.v1.ScheduledAccessSetup scheduled_access_setups = 1;  // repeated
}
message GetSectionsByCourseRequest {  // @2148065
  lis.resource.v1.CourseRef course = 1;
}
message GetSectionsByCourseResponse {  // @2149274
  lis.resource.v1.Section sections = 1;  // repeated
}
message GetStudentAssessmentAttemptsRequest {  // @2177939
  int64 org_id = 1;
  int64 provider_instance_id = 2;
  string external_course_id = 3;
  string external_resource_id = 4;
  lis.resource.v1.ExternalResourceType external_type = 5;
  int64 provider_user_id = 6;
  string provider_token = 7;
}
message GetStudentAssessmentAttemptsResponse {  // @2180809
  int32 allowed_attempts = 1;
  int32 current_attempts = 2;
  int32 extra_attempts = 3;
  lis.resource.v1.LastSubmission last_submission = 4;
}
message GetUserActivitiesRequest {  // @2162074
  int64 provider_instance_id = 1;
  int64 provider_user_id = 2;
  string user_email = 3;
}
message GetUserActivitiesResponse {  // @2163523
  lis.resource.v1.UserCoursesActivities activities = 1;
}
message GetUsersByAssignmentRequest {  // @2152082
  int64 provider_instance_id = 1;
  int64 course_id = 2;
  int64 assignment_id = 3;
}
message GetUsersByAssignmentResponse {  // @2154763
  lis.resource.v1.User users = 1;  // repeated
}
message GetUsersByCourseRequest {  // @2155947
  lis.resource.v1.CourseRef course = 1;
}
message GetUsersByCourseResponse {  // @2157298
  lis.resource.v1.AccessUser users = 1;  // repeated
  lis.resource.v1.UnknownUser unknown_users = 2;  // repeated
}
message GetUsersByGroupOrSectionRequest {  // @2158779
  lis.resource.v1.CourseRef course = 1;
  string provider_group_id = 2;
  string provider_section_id = 3;
}
message GetUsersByGroupOrSectionResponse {  // @2160386
  lis.resource.v1.AccessUser users = 1;  // repeated
  lis.resource.v1.UnknownUser unknown_users = 2;  // repeated
}
message GetWorkspaceActivitiesRequest {  // @2164767
  int64 provider_instance_id = 1;
}
message GetWorkspaceActivitiesResponse {  // @2166063
  lis.resource.v1.ActivityItem activities = 1;  // repeated
}
message Group {  // @2132692
  string provider_group_id = 1;
  string name = 2;
}
message HandInExamRequest {  // @2167203
  lis.resource.v1.ActivityRef activity = 1;
}
message HandInExamResponse { }  // empty @2168120
message LastSubmission {  // @2084415
  int32 attempt_number = 1;
  google.protobuf.Timestamp submitted_at = 2;
  string status = 3;
}
message LearningResource {  // @2076295
  int64 id = 1;
  int64 org_id = 2;
  lis.resource.v1.ActivityType activity_type = 3;
  google.protobuf.Timestamp created_at = 4;
}
message ListResourcesByOrgRequest {  // @2169148
  int64 org_id = 1;
}
message ListResourcesByOrgResponse {  // @2170338
  lis.resource.v1.FullResourceView resources = 1;  // repeated
}
message ReleaseUserRequest {  // @2171620
  int64 provider_instance_id = 1;
  lis.resource.v1.AccessUser user = 2;
}
message ReleaseUserResponse { }  // empty @2172654
message ResolveExternalRequest {  // @2173945
  int64 provider_instance_id = 1;
  string external_course_id = 2;
  string external_resource_id = 3;
}
message ResolveExternalResponse {  // @2175310
  lis.resource.v1.FullResourceView view = 1;
}
message ResourceBinding {  // @2078399
  int64 id = 1;
  int64 learning_resource_id = 2;
  int64 provider_instance_id = 3;
  string external_course_id = 4;
  string external_resource_id = 5;
  lis.resource.v1.ExternalResourceType external_type = 6;
  string external_url = 7;
}
message ResourcePolicy {  // @2080845
  int64 id = 1;
  int64 learning_resource_id = 2;
  bool lockdown = 3;
  bool clipboard_allowed = 4;
  string settings_json = 5;
  google.protobuf.Timestamp created_at = 6;
  google.protobuf.Timestamp updated_at = 7;
}
message ScheduledAccessSetup {  // @2089797
  int64 id = 1;
  string name = 2;
  google.protobuf.Timestamp available_from = 3;
  google.protobuf.Timestamp available_until = 4;
  google.protobuf.Timestamp due_at = 5;
  int64 created_by_user_id = 6;
  google.protobuf.Timestamp created_at = 7;
  google.protobuf.Timestamp updated_at = 8;
  lis.resource.v1.ScheduledAccessUser users = 9;  // repeated
  lis.resource.v1.SetupSchedulingStatus status = 10;
  string reason = 11;
  bool requires_attention = 12;
}
message ScheduledAccessTimesUpdate {  // @2190210
  google.protobuf.Timestamp available_from = 1;
  google.protobuf.Timestamp available_until = 2;
  google.protobuf.Timestamp due_at = 3;
}
message ScheduledAccessUser {  // @2093112
  int64 id = 1;
  int64 iam_user_id = 2;
  lis.resource.v1.UserSchedulingStatus status = 3;
  string reason = 4;
  google.protobuf.Duration time_extension = 5;
  google.protobuf.Timestamp handed_in_at = 6;
  google.protobuf.Timestamp released_at = 7;
  int64 released_by = 8;
}
message ScheduledAccessUsersUpdate {  // @2188667
  lis.resource.v1.AccessUser updated_users = 1;  // repeated
}
message Section {  // @2150463
  string provider_section_id = 1;
  string name = 2;
}
message UnknownUser {  // @2094704
  string email = 1;
}
message UpdateResourceLockForUserRequest {  // @2182686
  int64 learning_resource_id = 1;
  string user_id = 2;
  bool lock = 3;
}
message UpdateResourceLockForUserResponse { }  // empty @2184020
message UpdateScheduledAccessSetupRequest {  // @2185787
  int64 scheduled_access_setup_id = 1;
  string name = 2;
  lis.resource.v1.ScheduledAccessUsersUpdate users = 3;
  lis.resource.v1.ScheduledAccessTimesUpdate times = 4;
}
message UpdateScheduledAccessSetupResponse {  // @2187387
  lis.resource.v1.ScheduledAccessSetup scheduled_access_setup = 1;
}
message UpsertPolicyRequest {  // @2191883
  int64 learning_resource_id = 1;
  bool lockdown = 2;
  bool clipboard_allowed = 3;
  string settings_json = 4;
}
message UpsertPolicyResponse {  // @2193301
  lis.resource.v1.ResourcePolicy policy = 1;
}
message User {  // @2153473
  string id = 1;
  string email = 2;
  string name = 3;
}
message UserActivitiesInfo {  // @2097635
  int64 provider_user_id = 1;
  string email = 2;
}
message UserCoursesActivities {  // @2096201
  google.protobuf.Timestamp generated_at = 1;
  lis.resource.v1.UserActivitiesInfo user = 2;
  lis.resource.v1.CourseWithActivities courses = 3;  // repeated
}
```

### lis.user.v1 -- 7 messages, 0 enums

```proto
message CreateUserRequest {  // @2205588
  int64 iam_org_id = 1;
  lis.user.v1.Userinfo user_info = 2;
}
message CreateUserResponse { }  // empty @2206599
message RollbackLoginsRequest {  // @2209910
  int64 iam_org_id = 1;
  string external_id = 2;
  string email = 3;
}
message RollbackLoginsResponse { }  // empty @2211105
message UpdateUserRequest {  // @2207685
  int64 iam_org_id = 1;
  lis.user.v1.Userinfo user_info = 2;
}
message UpdateUserResponse { }  // empty @2208696
message Userinfo {  // @2204143
  string external_id = 1;
  string email = 2;
  string first_name = 3;
  string last_name = 4;
}
```

### lockdown.executionresult.v1 -- 17 messages, 3 enums

```proto
enum ExecutionPhase {   EXECUTION_PHASE_UNSPECIFIED = 0;   EXECUTION_PHASE_STARTED = 1;   EXECUTION_PHASE_STOPPED = 2; }  // lockdown.executionresult.v1.ExecutionPhase @2212070
enum ExecutionStatus {   EXECUTION_STATUS_UNSPECIFIED = 0;   EXECUTION_STATUS_SUCCESS = 1;   EXECUTION_STATUS_FAILURE = 2; }  // lockdown.executionresult.v1.ExecutionStatus @2212624
enum Severity {   SEVERITY_UNSPECIFIED = 0;   SEVERITY_DEBUG = 1;   SEVERITY_INFO = 2;   SEVERITY_WARNING = 3;   SEVERITY_ERROR = 4; }  // lockdown.executionresult.v1.Severity @2217134
message CreateExecutionResultRequest {  // @2224079
  int64 organization_id = 1;
  int64 user_id = 2;
  string exam_id = 3;
  string session_id = 4;
  int64 operation_id = 5;
  string operation_name = 6;
  google.protobuf.Timestamp executed_at = 7;
  lockdown.executionresult.v1.ExecutionPhase phase = 8;
  lockdown.executionresult.v1.ExecutionStatus status = 9;
  bool rollback = 10;
  string reason = 11;
}
message CreateExecutionResultResponse {  // @2226273
  lockdown.executionresult.v1.ExecutionResult execution_result = 1;
}
message DeleteExecutionResultRequest {  // @2227506
  int64 id = 1;
}
message DeleteExecutionResultResponse { }  // empty @2228595
message ExecutionResult {  // @2215504
  int64 id = 1;
  int64 organization_id = 2;
  int64 user_id = 3;
  string exam_id = 4;
  string session_id = 5;
  int64 operation_id = 6;
  string operation_name = 7;
  google.protobuf.Timestamp executed_at = 8;
  lockdown.executionresult.v1.ExecutionPhase phase = 9;
  lockdown.executionresult.v1.ExecutionStatus status = 10;
  bool rollback = 11;
  string reason = 12;
}
message GetAggregatedExecutionResultInfoRequest {  // @2230433
  int64 organization_id = 1;
  string exam_id = 2;
  google.protobuf.Timestamp execution_window_start = 3;
  google.protobuf.Timestamp execution_window_end = 4;
}
message GetAggregatedExecutionResultInfoResponse {  // @2232139
  lockdown.executionresult.v1.OperationExecutionResultAggregation operation_aggregates = 1;  // repeated
}
message GetExecutionResultLogsInternalRequest {  // @2236643
  string session_id = 1;
  int64 operation_id = 2;
  google.protobuf.Timestamp execution_window_start = 3;
  google.protobuf.Timestamp execution_window_end = 4;
}
message GetExecutionResultLogsInternalResponse {  // @2238310
  lockdown.executionresult.v1.LogEntry entries = 1;  // repeated
}
message GetExecutionResultLogsPublicRequest {  // @2240307
  string session_id = 1;
  int64 operation_id = 2;
  int64 org_id = 3;
  google.protobuf.Timestamp execution_window_start = 4;
  google.protobuf.Timestamp execution_window_end = 5;
}
message GetExecutionResultLogsPublicResponse {  // @2242046
  lockdown.executionresult.v1.LogEntry entries = 1;  // repeated
}
message GetExecutionResultsByIDsRequest {  // @2233481
  int64 ids = 1;  // repeated
}
message GetExecutionResultsByIDsResponse {  // @2234788
  lockdown.executionresult.v1.ExecutionResult execution_result = 1;  // repeated
}
message GetManyExecutionResultsWithFilterRequest {  // @2244383
  int64 organization_id = 1;
  int64 user_id = 2;
  string exam_id = 3;
  google.protobuf.Timestamp execution_window_start = 4;
  google.protobuf.Timestamp execution_window_end = 5;
  string session_id = 6;
}
message GetManyExecutionResultsWithFilterResponse {  // @2246258
  lockdown.executionresult.v1.ExecutionResult execution_results = 1;  // repeated
}
message LogEntry {  // @2218671
  google.protobuf.Timestamp timestamp = 1;
  lockdown.executionresult.v1.Severity severity = 2;
  string service_name = 3;
  bytes log_data = 4;
}
message OperationExecutionResultAggregation {  // @2220764
  int64 operation_id = 1;
  int64 succeeded_count = 2;
  int64 failed_count = 3;
  int64 failed_execution_result_ids = 4;  // repeated
}
```

### lockdown.lifecycle.v1 -- 27 messages, 0 enums

```proto
message ApplicableOperation {  // @2257139
  int64 id = 1;
  int64 definition_id = 2;
  string definition_name = 3;
  int64 organization_id = 4;
  int64 student_id = 5;
  string name = 6;
  string description = 7;
  bool is_enabled = 8;
  bool is_archived = 9;
  bool require_administrator = 10;
  google.protobuf.Timestamp valid_from = 11;
  google.protobuf.Timestamp valid_to = 12;
  google.protobuf.Timestamp created_at = 13;
  google.protobuf.Timestamp updated_at = 14;
  lockdown.parameter.v1.Parameter parameters = 15;  // repeated
  int64 overrides_operation_id = 16;
  lockdown.opplatform.v1.OperationPlatform operation_platform = 17;  // repeated
}
message ArchiveOperationRequest {  // @2271878
  int64 operation_id = 1;
}
message ArchiveOperationResponse {  // @2273088
  lockdown.lifecycle.v1.Operation operation = 1;  // repeated
}
message CreateDefaultOperationRequest {  // @2275027
  int64 definition_id = 1;
  string description = 2;
  string name = 3;
  bool require_administrator = 4;
  lockdown.parameter.v1.Parameter parameters = 5;  // repeated
}
message CreateDefaultOperationResponse {  // @2276653
  lockdown.lifecycle.v1.Operation operation = 1;
}
message DeleteOperationRequest {  // @2277816
  int64 id = 1;
}
message DeleteOperationResponse { }  // empty @2278827
message GetAllApplicableOperationsRequest {  // @2280165
  int64 organization_id = 1;
  int64 student_id = 2;
}
message GetAllApplicableOperationsResponse {  // @2281607
  lockdown.lifecycle.v1.ApplicableOperation operations = 1;  // repeated
}
message GetAllDefaultOperationsRequest { }  // empty @2282720
message GetAllDefaultOperationsResponse {  // @2283911
  lockdown.lifecycle.v1.Operation operations = 1;  // repeated
}
message GetLockdownConfigOfflineRequest {  // @2285185
  int64 organization_id = 1;
}
message GetLockdownConfigOfflineResponse {  // @2286473
  lockdown.lifecycle.v1.LockdownConfig config = 1;
}
message GetLockdownConfigRequest {  // @2287840
  int64 organization_id = 1;
  int64 student_id = 2;
}
message GetLockdownConfigResponse {  // @2289144
  lockdown.lifecycle.v1.LockdownConfig config = 1;
}
message GetManyOperationsWithFilterRequest {  // @2291390
  int64 organization_id = 1;
  int64 student_id = 2;
  google.protobuf.Timestamp valid_from = 3;
  google.protobuf.Timestamp valid_to = 4;
  bool is_enabled = 5;
  bool is_archived = 6;
  string name = 7;
}
message GetManyOperationsWithFilterResponse {  // @2293107
  lockdown.lifecycle.v1.Operation operations = 1;  // repeated
}
message GetManyOrgOperationsWithFilterRequest {  // @2294638
  int64 organization_id = 1;
  bool archived = 2;
}
message GetManyOrgOperationsWithFilterResponse {  // @2296115
  lockdown.lifecycle.v1.Operation operations = 1;  // repeated
}
message LockdownConfig {  // @2265476
  lockdown.lifecycle.v1.ApplicableOperation applicable_operations = 1;  // repeated
  lockdown.opdefinition.v1.OperationDefinition operation_definitions = 2;  // repeated
}
message Operation {  // @2269422
  int64 id = 1;
  int64 definition_id = 2;
  string definition_name = 3;
  int64 organization_id = 4;
  int64 student_id = 5;
  string name = 6;
  string description = 7;
  bool is_enabled = 8;
  bool is_archived = 9;
  bool require_administrator = 10;
  google.protobuf.Timestamp valid_from = 11;
  google.protobuf.Timestamp valid_to = 12;
  google.protobuf.Timestamp created_at = 13;
  google.protobuf.Timestamp updated_at = 14;
  lockdown.parameter.v1.Parameter parameters = 15;  // repeated
  int64 overrides_operation_id = 16;
  int64 default_operation_id = 17;
}
message OverrideOperationRequest {  // @2299090
  int64 definition_id = 1;
  int64 overrides_operation_id = 2;
  int64 organization_id = 3;
  int64 student_id = 4;
  string description = 5;
  string name = 6;
  bool is_enabled = 7;
  bool require_administrator = 8;
  google.protobuf.Timestamp valid_from = 9;
  google.protobuf.Timestamp valid_to = 10;
  lockdown.parameter.v1.Parameter parameters = 11;  // repeated
}
message OverrideOperationResponse {  // @2301188
  lockdown.lifecycle.v1.Operation operation = 1;
}
message RestoreArchivedOperationRequest {  // @2302423
  int64 operation_id = 1;
}
message RestoreArchivedOperationResponse {  // @2303735
  lockdown.lifecycle.v1.Operation operation = 1;  // repeated
}
message UpdateOperationRequest {  // @2306173
  int64 id = 1;
  string description = 2;
  string name = 3;
  bool is_enabled = 4;
  bool require_administrator = 5;
  google.protobuf.Timestamp valid_from = 6;
  google.protobuf.Timestamp valid_to = 7;
  lockdown.parameter.v1.Parameter parameters = 8;  // repeated
}
message UpdateOperationResponse {  // @2307850
  lockdown.lifecycle.v1.Operation operation = 1;
}
```

### lockdown.opdefinition.v1 -- 9 messages, 0 enums

```proto
message CreateOperationDefinitionRequest {  // @2309457
  string name = 1;
  string description = 2;
  lockdown.parameter.v1.ParameterDefinition parameter_definitions = 3;  // repeated
}
message CreateOperationDefinitionResponse {  // @2311009
  lockdown.opdefinition.v1.OperationDefinition operation_definition = 1;
}
message DeleteOperationDefinitionRequest {  // @2312298
  int64 id = 1;
}
message DeleteOperationDefinitionResponse { }  // empty @2313432
message GetAllOperationDefinitionsRequest { }  // empty @2314478
message GetAllOperationDefinitionsResponse {  // @2315756
  lockdown.opdefinition.v1.OperationDefinition operation_definitions = 1;  // repeated
}
message OperationDefinition {  // @2263763
  int64 id = 1;
  string name = 2;
  string description = 3;
  google.protobuf.Timestamp created_at = 4;
  google.protobuf.Timestamp updated_at = 5;
  lockdown.parameter.v1.ParameterDefinition parameter_definitions = 7;  // repeated
}
message UpdateOperationDefinitionRequest {  // @2317395
  int64 id = 1;
  string description = 2;
  string name = 3;
}
message UpdateOperationDefinitionResponse {  // @2318868
  lockdown.opdefinition.v1.OperationDefinition operation_definition = 1;
}
```

### lockdown.opplatform.v1 -- 9 messages, 0 enums

```proto
message AssignOperationPlatformsToOperationRequest {  // @2320403
  int64 operation_id = 1;
  int64 platform_ids = 2;  // repeated
}
message AssignOperationPlatformsToOperationResponse { }  // empty @2321738
message CreateOperationPlatformRequest {  // @2323567
  int64 operation_id = 1;
  int64 os_id = 2;
  int64 arch_id = 3;
  int32 min_version = 4;
  int32 max_version = 5;
}
message CreateOperationPlatformResponse {  // @2325211
  lockdown.opplatform.v1.OperationPlatform operation_platform = 1;
}
message DeleteOperationPlatformRequest {  // @2326479
  int64 id = 1;
}
message DeleteOperationPlatformResponse { }  // empty @2327587
message GetAllOperationPlatformsRequest { }  // empty @2328607
message GetAllOperationPlatformsResponse {  // @2329822
  lockdown.opplatform.v1.OperationPlatform platforms = 1;  // repeated
}
message OperationPlatform {  // @2252782
  int64 id = 1;
  int64 operation_id = 2;
  int64 os_id = 3;
  int64 arch_id = 4;
  int32 min_version = 5;
  int32 max_version = 6;
}
```

### lockdown.parameter.v1 -- 5 messages, 1 enums

```proto
enum ParameterType {   PARAMETER_TYPE_UNSPECIFIED = 0;   PARAMETER_TYPE_STRING = 1;   PARAMETER_TYPE_INT = 2;   PARAMETER_TYPE_BOOL = 3; }  // lockdown.parameter.v1.ParameterType @2259147
message IntegerArray {  // @2250968
  int32 values = 1;  // repeated
}
message Parameter {  // @2247546
  int32 order = 1;
  lockdown.parameter.v1.ParameterValue value = 2;
}
message ParameterDefinition {  // @2261188
  int64 id = 1;
  string name = 2;
  string description = 3;
  int32 order = 4;
  lockdown.parameter.v1.ParameterType parameter_type = 5;
  bool array = 6;
  bool optional = 7;
}
message ParameterValue {  // @2248640
  string string_val = 1;
  int32 integer_val = 2;
  bool boolean_val = 3;
  lockdown.parameter.v1.StringArray string_array_val = 4;
  lockdown.parameter.v1.IntegerArray integer_array_val = 5;
}
message StringArray {  // @2249971
  string values = 1;  // repeated
}
```

### lockdown.platform.v1 -- 10 messages, 0 enums

```proto
message CreateOperatingSystemArchitectureRequest {  // @2333956
  string name = 1;
}
message CreateOperatingSystemArchitectureResponse {  // @2335428
  lockdown.platform.v1.OperatingSystemArchitecture operating_system_architecture = 1;
}
message CreateOperatingSystemRequest {  // @2336706
  string name = 1;
}
message CreateOperatingSystemResponse {  // @2337983
  lockdown.platform.v1.OperatingSystem operating_system = 1;
}
message GetAllOperatingSystemArchitecturesRequest { }  // empty @2341406
message GetAllOperatingSystemArchitecturesResponse {  // @2342753
  lockdown.platform.v1.OperatingSystemArchitecture operating_system_architectures = 1;  // repeated
}
message GetAllOperatingSystemsRequest { }  // empty @2339053
message GetAllOperatingSystemsResponse {  // @2340219
  lockdown.platform.v1.OperatingSystem operating_systems = 1;  // repeated
}
message OperatingSystem {  // @2332582
  int64 id = 1;
  string name = 2;
}
message OperatingSystemArchitecture {  // @2331221
  int64 id = 1;
  string name = 2;
}
```

## 4. Native-side vocabulary (Rust serde/prost reflection strings)

These are **not** protobuf descriptors — names only, no field numbers/types. They corroborate the frontend schemas above and document native-only structures.

### 4.1 Lockdown packages corroborated on the native side — VERIFIED

Message-type name strings (prost `Name`/`Debug` format strings `struct <fqname>`):

- digiexam.exe @0x0221A27B–0x0221A42C (`strings_digiexam.txt` L154139–154149): `lockdown.lifecycle.v1.ApplicableOperation`, `lockdown.lifecycle.v1.LockdownConfig`, `lockdown.opdefinition.v1.OperationDefinition`, `lockdown.parameter.v1.ParameterValue`, `lockdown.parameter.v1.StringArray`, `lockdown.parameter.v1.IntegerArray`, `lockdown.parameter.v1.Parameter`, `lockdown.parameter.v1.ParameterDefinition`, `lockdown.opplatform.v1.OperationPlatform` — same package/type set as the frontend descriptors.
- service.exe @0x00DBA138 (@0x00DBA2F8, @0x00DBA6F9) (`strings_service.txt` L154139–154149, L147222): same set.

Serde key clusters (dual-case `snake_case`+`camelCase` pairs, i.e. wire name + JSON name) — VERIFIED:

| Cluster | digiexam.exe | Matches frontend descriptor |
|---|---|---|
| ParameterValue: `string_val`/`stringVal`, `integer_val`/`integerVal`, `boolean_val`/`booleanVal`, `string_array_val`/`stringArrayVal`, `integer_array_val`/`integerArrayVal` | @0x01FD0200, @0x01FD025B | `lockdown.parameter.v1.ParameterValue` fields 1–5 (oneof) |
| OperationPlatform: `id`, `operation_id`/`operationId`, `os_id`/`osId`, `arch_id`/`archId`, `min_version`/`minVersion`, `max_version`/`maxVersion` | @0x01FD0338 | see §6.3 — frontend has `min_version` only |
| ApplicableOperation/Operation: `definition_id`, `definition_name`, `organization_id`, `student_id`, `name`, `description`, `is_enabled`, `is_archived`, `require_administrator`, `valid_from`, `valid_to`, `created_at`, `updated_at`, `parameters`, `overrides_operation_id`, `operation_platform` | @0x01B1CEF0, @0x01B1CF6F, @0x01FD0438, @0x01FD04C2, @0x01FD0890 | `lockdown.lifecycle.v1.ApplicableOperation` fields 1–16 (+17) |
| `parameterDefinitions` + `lockdown.lifecycle.v1.ApplicableOperation` (LockdownConfig) | @0x01FD0890 | `lockdown.lifecycle.v1.LockdownConfig` |
| `RefreshTokenResponse` + `access_token` + `access_token_expires_at` | @0x01E7DA48 | `assignment.auth.v1.RefreshTokenResponse` fields 2, 4 |
| `CreateLockdownSystemSnapshotUploadResponse` + `snapshot_id` + `upload_url` | @0x01FD0A6A | `assignment.snapshot.v1.CreateLockdownSystemSnapshotUploadResponse` fields 1, 2 |

### 4.2 Native `SessionInfo` and IPC payload vocabulary — VERIFIED (not protobuf)

- `struct SessionInfo with 4 elements` (serde reflection, service.exe `strings_service.txt` L146669, @0x00C7FB28 cluster) with field names `organization_id`, `user_id`, `exam_id`, `is_proctored` — a 4-field native struct (snapshot-collector session context; "Session info was not set for snapshot collector!" — L147509).
- Same cluster (digiexam.exe `strings_digiexam.txt` L416068): serde untagged-enum error context `data did not match any variant of untagged enum Content` followed by `organization_id user_id exam_id is_proctored access_token refresh_token id_token expires_in client_session_id` and `tuple variant ExecutorIpcRequest::RequestLockdownStart with 2 elements` — the Tauri/IPC `Content` payload vocabulary (auth-token fields + SessionInfo + IPC commands).
- IPC command/event names (same clusters, L416068 / L147111 / service.exe L147634): `UpdateFromClient`, `ClientStarted`, `client_session_id`, `RegionChanged`, `new_region`, `SessionChanged`, `Ping`, `RequestLockdownStart`, `RequestLockdownStartWithDefinitions`, `RequestLockdownStop`, `RollbackOperations`, `LockdownStopComplete`, `NotifyAbort`, `OperationRequiresUserAction`, `ManualProcessKillingRequired`, `Started`, `has_rollback`, `Stopped`, `Error`, `ActionRequired`. These are native IPC names (ExecutorIpcRequest variants), **not** verified gRPC methods (see §6.2).
- Native `ExecutionResult` struct (L416068, @0x01B1BE32): `operation_id`, `operation_name`, `status`, `executed_at` (+ state names above) — a native rollback-log/IPC struct whose vocabulary mirrors but does not equal proto `lockdown.executionresult.v1.ExecutionResult` (which additionally has `id`, `organization_id`, `user_id`, `exam_id`, `session_id`, `phase`, `rollback`).

### 4.3 System-snapshot schema (upload payload) — names VERIFIED, nesting STRONGLY INFERRED

String cluster `strings_digiexam.txt` L416064–416074 (digiexam.exe @0x01FCFA14, @0x01FCFB54) and `strings_service.txt` L147114 (@0x00C7FD1D). Individual field names are VERIFIED; the tree below is the STRONGLY INFERRED grouping (adjacency + semantics; class names `HardwareGraphic`, `HardwareStorage`, `NetworkInterface`, `VirtualizationComponent`, `VirtualizationEvidence`, `MemoryModule` are VERIFIED type names in the same cluster):

- top level: `schema_version`, `collector_version`, `collected_at`, `platform`, `operating_system`, `hardware`, `software`, `network`, `virtualization`
- os/system group: `architecture`, `process_architecture`, `execution_environment`, `name`, `version`, `build`, `kernel_version`, `secure_boot_enabled`, `system_integrity_protection_enabled`, `hypervisor_present`
- firmware group: `manufacturer`, `model_identifier`, `model_name`, `family`, `platform_type`, `boot_rom_version`, `platform_firmware_version`
- processor group: `model`, `physical_core_count`, `logical_processor_count`, `performance_core_count`, `efficiency_core_count`, `virtualization_supported`, `hypervisor_vendor`
- memory group: `total_bytes`, `memory_model`, `type`, `configured_speed_mhz`, `module_count`, `channel_mode`, `modules` → module: `size_bytes`, …
- `HardwareGraphic`: `vendor_id`, `device_id`, `classification`, `metal_supported`
- `HardwareStorage`: `capacity_bytes`, `media_type`, `bus_type`, `location`, `virtual`, `controller_provider`
- `NetworkInterface`: `default_interface`, `other_interfaces`, `display_name`, `state`, `provider`, `related_platform`, `driver_bundle`, `addresses`, `default_gateways`, `dns_servers`
- virtualization group: `installed_virtualization_software`, `host_virtualization_capability`, `matched_virtualization_components` → `VirtualizationComponent`: `category`; `VirtualizationEvidence`: `summary`, `evidence`, `detected_platforms`, `source`, `value`

Upload handshake (protobuf, §3 `assignment.snapshot.v1`): `CreateLockdownSystemSnapshotUploadRequest { snapshot_id, exam_id, student_id, session_id, schema_version, content_size_bytes, crc32c, platform }` → `CreateLockdownSystemSnapshotUploadResponse { snapshot_id, upload_url }` — i.e. the snapshot itself goes to a pre-signed `upload_url`, not through the gRPC message.

### 4.4 Toolbox-proxy schema drift — VERIFIED (older iteration)

`toolbox-ipc-proxy.exe` (`strings_toolbox.txt` L12749–12759, L13372–13377; raw offsets @0x0B92B0–0x0B9B2C) embeds the same serde vocabulary under a **different package layout**:

- `lockdown.lifecycle.v1.{ApplicableOperation, Parameters, ParameterValue, IntegerArray, StringArray}` (@0x0B92B0, @0x0C1A34–0x0C1A3B) — parameter machinery under `lifecycle`, including a type `Parameters` that does not exist in the frontend schema; the current binaries place these under `lockdown.parameter.v1` / `lockdown.lifecycle.v1` respectively.
- `lockdown.opplatform.v1.OperationPlatform` **with** `max_version`/`maxVersion` (@0x0B9B2C) — the frontend descriptor has only `min_version` (field 5).
- IPC command names: `Ping`, `LockdownStarting`, `UpdateFromClient`, `RunOperation`, `StopOperation`, `RollbackOperations`, `NotifyInstallerRequest…` (@0x0B8651 cluster); channel name `digiexam_toolbox_service_ipc` (@0x0B72D8); pipe framing in `native\libs\lockdown-ipc\src\framing.rs` (dev-machine path `C:\Users\AndreVennberg\RustroverProjects\frontend\…` @0x0BA460 — different build environment than the CI-built main binaries, consistent with a separately-versioned component).

## 5. Methodology & limits

- **Reconstructed, not recovered source.** Everything in §2–§3 is derived from *embedded reflection/descriptor strings* of compiled artifacts. The original `.proto` files were not accessed (they are not shipped); statement order, options, comments, imports, and any server-only message/field details are unknown.
- **Fidelity split.** Frontend-known messages: field numbers, types, repetition, and map/enum details are available (protobuf-es descriptors survive minification). Native-only vocabulary (§4): field *names* only — numbering, types, and message boundaries are not recoverable from serde/prost strings.
- **Client-side view.** The frontend descriptors describe what the *client* compiled in. Server-side-only messages, fields, or whole RPCs would not appear here. Completeness is not guaranteed in either direction (a field can also be dead code client-side).
- **String-pool adjacency.** The Rust-side clusters are runs of deduplicated strings in the binary's string pool; attribution of a field name to a struct relies on adjacency and semantic fit (STRONGLY INFERRED), and adjacent strings may merge with overlaps in the dump (e.g. `hypervisor_preseure_boot_enabledsecure_boot_enab…` in `strings_digiexam.txt` L416064–416067 are fragmentation artifacts of the same name set).
- **Minified names.** A handful of message references in the bundle are minified duplicates (`User$1`, `User$3`, `Group$1`, `Status$1`); they were resolved by proximity to their `typeName` registrations and are included under their real names.
- **No execution, no traffic.** Static analysis of files on disk only; raw offsets were obtained by byte-scanning `D:\Digiexam` binaries read-only. Nothing was patched or run; the lockdown configuration surface is documented as interface description only.
- **Verification command shape.** Any entry can be re-verified: frontend — `IndexOf('"assignment.exam.v1.ExamDetails"')` in the bundle; native — ASCII search at the given raw offsets in the named binary.

## 6. Contradictions & uncertainties

1. **`SaveTextualAnswer` kind — correction to doc 10.** Earlier summarized as "(server-streaming)"; the frontend descriptor states `MethodKind.ClientStreaming` (@1665998) — the client streams answer updates up and receives one response. `DownloadFile` is the server-streaming RPC (@1667649); `UploadFile` and `StreamLog` are client-streaming.
2. **`LockdownStart` / `LockdownStop` / `RollbackOperations` as LockdownService gRPC methods — NOT VERIFIED.** No tonic method-path string exists for them; they appear as `ExecutorIpcRequest` variants / IPC command names (§4.2) and in the toolbox proxy (§4.4). Doc 10 §2's "phase-1 (strings)" rows should be read as IPC commands unless a tonic path is found; the only verified tonic paths for `lockdown.service.v1.LockdownService` are `GetLockdownConfig` here (and the 31 frontend grpc-web methods, none of which is `LockdownStart`).
3. **`max_version` on `OperationPlatform`.** Present in the native serde clusters (digiexam.exe @0x01FD0338, toolbox @0x0B9B2C) but absent from the frontend descriptor (which stops at `min_version = 5`). Either the wire schema has a field 6 the current frontend predates, or the native struct carries a legacy/unused field. UNVERIFIED which.
4. **Package drift between binaries** (§4.4): `lockdown.lifecycle.v1.Parameters`/`ParameterValue`/`IntegerArray`/`StringArray` (toolbox) vs `lockdown.parameter.v1.*` (frontend + digiexam.exe/service.exe). The toolbox-ipc-proxy appears to lag the main binaries.
5. **I/O types for non-Assignment services** are not inlined in the frontend service definitions; request/response pairing by naming convention is STRONGLY INFERRED. A few descriptors pair requests with `assignment.service.v1.EmptyMessage`.
6. **Field-name attribution in the snapshot tree** (§4.3) is adjacency-based; nested grouping may differ in the real struct. The `metal_supported` field (Metal = macOS) indicates a cross-platform schema even though only the Windows collector ships here.
7. **`operating_system` vs `os`** both appear at the snapshot top-level cluster — possibly two schema generations side by side in one binary; attribution UNVERIFIED.
8. **WEB/WASM frontend builds** were not shipped/analyzed; the 199-RPC surface is the native build's compiled-in subset (doc 08 §5).

## 7. Evidence index

- Frontend bundle: `source/recovered-assets/frontend/assets/index-COO2fUbz.js` (char offsets quoted inline).
- String dumps: `source/recovered-assets/strings/strings_digiexam.txt` (L146156–146157, L269077, L416064–416101), `strings_service.txt` (L146156–146157, L146665–146669, L147059–147114, L147222, L147634–147646, L154139–154149), `strings_toolbox.txt` (L12749–12759, L13372–13377).
- Binaries (read-only offset pinning): `D:\Digiexam\digiexam.exe`, `D:\Digiexam\service.exe`, `D:\Digiexam\toolbox-ipc-proxy.exe` (raw offsets quoted inline).
- Related: [`10-backend-protocols.md`](../phase-2/10-backend-protocols.md) (transports, endpoints), [`08-frontend-architecture.md`](../phase-2/08-frontend-architecture.md) (bundle provenance), [`09-windows-integration.md`](../phase-2/09-windows-integration.md) (toolbox topology).
