# Capture, listen, review

## Capture and audit

Capture every declared object state, independent continuous-control interventions, near/far approaches at matched source controls, listener binding and physical event-to-onset timing. Run satie_audit_playthrough with script + scene + telemetry; a capture too large to pass inline goes up once through satie_asset { kind: "capture" } and is audited by its captureId. Legacy captures without causal evidence remain unverified. PCM onset estimates verify scheduling plus source-file lead-in, not actual device output latency. Record and listen to gameplay separately, then pass scene + behaviorTelemetry to satie_review_release; runtimeOk assertions cannot replace behavior evidence. Local capture is not training consent. Only call satie_record_episode after explicit developer consent to contribute.

## Listening corrections

satie_listen accepts up to three rejected WAV references plus the actual human/agent reason (combined audio payload is bounded). Feed the objection and audible counterexample into the next review and composition brief. It returns exact conditioning hashes. Preserve the downloaded asset hash separately from any converted audition-WAV hash, with conversion provenance; the listener hashes its WAV, while release decisions refer to verified original asset bytes. A model may accept a bad sound: preserve known human rejections by exact asset hash, and do not erase them with a later model pass. Reference-conditioned rejection of a known example is regression coverage, not evidence of held-out generalization or trained model improvement.

Listen with files, not base64: keep each WAV with satie_asset kind "audio" and pass audioId; references are audioIds with the reason.
