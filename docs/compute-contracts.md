# CPU and local-model contracts

The following describes inspected public implementation at the [pinned baseline](architecture-and-projects.md). Existing [Camel](camel-kie-cpu-dag.md), [Chrome2api](chrome2api.md), and [fleet](fleet-capability-invariants.md) guides provide context; the distinctions below prevent configuration, receipts, and model output from being mistaken for verified behavior.

## Java package contracts

All production classes below are in `com.synexia.codexpro.dag` under [the main source root](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/CamelCpuDag.java). The Maven artifact is separately built from the Node package.

| Class | Inputs, outputs, state, and failures |
| --- | --- |
| [CamelCpuDag](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/CamelCpuDag.java) | Copies/validates the capability registry, creates policy/pool/context/routes/producer, admits a request, fans out registered text functions, seals ordered receipts. Implements close; callers own lifecycle. Constructor or close failures are not protected by a comprehensive rollback/finally chain |
| [CamelCpuDagMain](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/CamelCpuDagMain.java) | Reads all stdin, parses one Jackson request, executes with try-with-resources, prints one JSON receipt. Admission bounds do not prevent allocating an arbitrarily large standalone stdin before parse |
| [DagRequest](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/DagRequest.java), [DagItem](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/DagItem.java) | Records carrying planId/items and id/capability/payload. Validation is in admission, not validating record constructors; the request is a list, not a caller-defined edge/dependency graph |
| [DagLimits](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/DagLimits.java) | Positive item/byte/queue/worker limits. Defaults: 512 items, 65,536 UTF-8 bytes/item, 4,194,304 bytes total, queue 64/capability, workers `max(1,min(8,floor(0.8*availableProcessors)))` |
| [DagAdmission](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/DagAdmission.java) | Mutable policy fact with request/capability/budget/endpoint flags and decision/reason; starts pending and is not an independently trusted authorization receipt |
| [DagPolicy](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/DagPolicy.java) | Loads and compiles bundled DRL, rejects compilation errors, hashes exact policy text, and synchronizes stateless-session evaluation. Unknown/pending decisions do not become allow by default |
| [StableFanIn](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/StableFanIn.java) | Aggregates asynchronous outputs for subsequent ordinal sorting; parallel completion order is not output order |
| [Hashing](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/Hashing.java) | UTF-8 SHA-256 and length-framed/domain-separated aggregate roots. Equality is a deterministic representation check, not a signature |
| [DagOutput](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/DagOutput.java), [DagReceipt](../camel-cpu-dag/src/main/java/com/synexia/codexpro/dag/DagReceipt.java) | Output ordinal/identity/content/hash and aggregate roots/limits/outputs. Caller interpretation must preserve policy/request/output distinctions |

### Admission and route lifecycle

The built-in registry contains `inventory.identity`, `inventory.canonicalize` (CRLF→LF and trim), `text.uppercase` (Locale.ROOT), and `hash.sha256`. A custom registry can replace text functions, but the capability-name list in a plan root does not fingerprint custom function bytecode.

Before routing any items, admission requires nonblank plan ID, nonempty bounded items, valid item/capability identifiers, nonnull payloads, per-item bytes, and an overflow-checked total. Duplicate item IDs are not rejected; ordinal is the ordering discriminator. The [DRL](../camel-cpu-dag/src/main/resources/com/synexia/codexpro/dag/dag-admission.drl) denies planner endpoint URIs, shell prefixes, unknown capabilities, and excess counts/bytes. Normal request admission supplies the endpoint flag as false because the request schema does not include arbitrary endpoints.

The route admits → splits with a fixed executor/stop-on-exception → dispatches to registry-derived SEDA endpoints → aggregates → sorts by input ordinal → seals. Each capability endpoint has a bounded queue with producer blocking and its own concurrent-consumer setting. The worker count is therefore not a global cap on all Camel/SEDA threads across all capabilities. Functions must return nonnull output within per-item size limits. A rejected request is not partially admitted; a function failure during execution is still different from transactional rollback of arbitrary custom-function side effects.

Close stops producer, context, and workers sequentially. Reusing one Java object is a different lifecycle from the Node adapter, which starts a JVM per execution. Tests inspect stable order/replay, rejection, and payload binding; they do not prove every custom function deterministic or side-effect free.

### Node/JVM boundary

[camelDagOps.ts](../src/camelDagOps.ts) exposes contract inspection and execution. Contract inspection reads the JAR/policy and probes `java -version`, so it is not a pure constant getter. Execution validates a bounded JSON object, launches `java -jar`, limits each output stream to two million bytes, and clamps the request timeout to 2–120 seconds. It inherits the process environment. Missing Java/JAR, launch/exit failures, output overflow, malformed JSON, or invalid receipt shape fail.

The Node receipt validator checks that the parsed value is a non-array object, has the exact receipt schema, and has a receiptRoot matching 64 lowercase hexadecimal characters. It does not validate all other receipt fields or independently recompute the other roots/hashes. Plan/request/policy/output roots serve different purposes and do not authenticate a remote principal. [fleetInvariants.ts](../src/fleetInvariants.ts) returns a declarative contract; it does not implement a distributed fleet scheduler or automatically enforce each stated invariant.

## Chrome2api completion adapter

[chrome2ApiContract.ts](../src/chrome2ApiContract.ts) and [chrome2ApiOps.ts](../src/chrome2ApiOps.ts) target the `chrome-gemini-nano` model at `http://127.0.0.1:11435/v1` by default. The configured URL must use HTTP and literal `127.0.0.1` or `[::1]`, exact `/v1`, and no credentials/query/hash. `localhost` is rejected by this contract. Redirects are rejected. This adapter does not acquire credentials, start Chrome, install models, accept media, or open supplied local file paths.

Completion accepts prompt/system text, max tokens, temperature, and timeout. Bounds are 65,536 prompt bytes, 16,384 system bytes, 81,920 combined bytes, 262,144 response bytes, 1–2,048 tokens (default 256), temperature 0–2 (default 0.2), and 2–120 seconds. It validates model identity and first-choice text content before returning a receipt. Status probes `/models` with a shorter default timeout and reports availability/errors; a model advertisement is not proof inference succeeds.

Canonical roots frame/domain-separate strings. In particular, inspect the actual `canonicalRoot` call when interpreting a response root rather than assuming a description string means raw SHA-256 of content. Receipts bind represented inputs/outputs; they are not signed, semantic validation, or proof that model output is safe to execute.

## Summarizer compatibility and cache behavior

[chromeSummarizerContract.ts](../src/chromeSummarizerContract.ts) and [chromeSummarizerOps.ts](../src/chromeSummarizerOps.ts) provide a batch compatibility layer, not the browser's native Summarizer object/lifecycle/streaming implementation. Supported type/format/length/preference/language enums are declared in the contract. Preference is advisory. The implementation supports key-points/tldr/teaser/headline, markdown/plain-text, short/medium/long, and the declared en/ja/es/de/fr language set.

Source text is bounded to 60,000 UTF-8 bytes and shared/per-request context to 4,096 bytes each. Prompts instruct the model to treat source as untrusted data and use temperature zero with requested output targets. Instructions do not enforce semantic obedience, word counts, language, or factual accuracy. Output authority remains candidate-only.

The bounded summary cache uses endpoint/source/options identity, clones results, joins identical in-flight work, and retains up to 128 entries/eight million estimated JSON bytes. Timeout is not part of the identity. A joined/hit request can therefore reuse work begun with a different caller timeout. Failures are not cached. Cache hits demonstrate reuse of a prior result, not a fresh model call or current backend health.

## Document and corpus lifecycle

[chromeDocumentSummarizer.ts](../src/chromeDocumentSummarizer.ts) accepts text up to two million bytes, splits into exact-reconstructing UTF-8 chunks (4,096–48,000 target bytes, at most 64), prefers paragraph/newline/punctuation boundaries late enough in the chunk, and avoids splitting a code point. It maps chunks sequentially using a long plain-text tldr, then reduces until the aggregate fits the reduction bound, with a depth-six/convergence guard. Final options are separate from intermediate summaries. Returned chunk manifests omit source text while retaining identities/offsets/hashes. A per-provider-call timeout is not an end-to-end workflow deadline.

[chromeCorpusSummarizer.ts](../src/chromeCorpusSummarizer.ts) accepts at most 64 uniquely identified documents, two million bytes each and eight million bytes total. It sorts IDs, groups exact content hashes with a text collision check, summarizes unique content sequentially, and retains aliases/identities in the receipt. This deduplicates provider work in memory; it does not remove or rewrite duplicate files. The helper itself accepts strings and does no workspace file traversal.

The server's `fabric chrome_summarize_document` reads one guarded text file and explicitly checks UTF-8 round-trip validity. Corpus mode accepts a case-sensitive top-level basename glob (default `*AI*.md`), rejects separators/`**`, bounds the matching list, validates every file and total bytes, then calls the corpus helper. That is a narrower source-selection contract than arbitrary recursive globbing. A successful hash/chunk manifest still does not certify summary truth.

[scripts/chrome-document-corpus-check.mjs](../scripts/chrome-document-corpus-check.mjs) is a separate local manifest checker: it reads top-level `*AI*.md`, validates UTF-8, computes chunk/content/corpus roots, and prints duplicate groups. It does not invoke the model or deduplicate disk content. No corpus command or private document was used in this documentation pass.

## Test evidence and next checks

The sidecar main/test namespace contains [CamelCpuDagTest](../camel-cpu-dag/src/test/java/com/synexia/codexpro/dag/CamelCpuDagTest.java), [DagPolicyTest](../camel-cpu-dag/src/test/java/com/synexia/codexpro/dag/DagPolicyTest.java), and [CamelDslCompileProbe](../camel-cpu-dag/src/test/java/com/synexia/codexpro/dag/CamelDslCompileProbe.java). The first two have three JUnit tests each; the probe is source-level DSL compilation coverage, not a separately annotated runtime test.

Node smoke files exercise fake-provider completion/summary/document/corpus paths and sidecar contracts; their exact commands and unexecuted status are in [coverage](documentation-coverage.md). Future verification should distinguish fixture protocol tests, real Chrome/model behavior, Java receipt recomputation, global thread counts, nonconvergent model reductions, custom-function side effects, and input-memory bounds. None is silently inferred from the existence of a receipt field.
