# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.8.3] - 2023-07-18
### Fixed
- Fix regression issue that prevented persistence of custom metadata json in DynamoDB call record
- Update QnAbot submodule branch/commit to pull in deployment fixes from QnAbot v5.3.5

## [0.8.2] - 2023-07-11
### Fixed
- Fix stack update problems related to Chime Voice Connector configuration when updating to v0.8.1 from v0.8.0 with Chime Call Analytics enabled

## [0.8.1] - 2023-07-10
### Fixed
- Add compatibility with [Start Call Processing](https://github.com/aws-samples/amazon-transcribe-live-call-analytics/blob/develop/lca-chimevc-stack/StartCallProcessingEvent.md#start-call-processing-for-a-in-progress-call) feature, when using Chime SDK Call Analytics audio processor.

## [0.8.0] - 2023-05-30

### Added

- Generative transcript summarization capability now supports Anthropic Claude third party API with 100K token limit (eliminating transcript length limitations). In a later release we will support Anthropic's models natively in AWS via the new [Amazon Bedrock](https://aws.amazon.com/bedrock/) service. See [Transcript Summarization](./lca-ai-stack/TranscriptSummarization.md#anthropic)
- Generative AI for Agent Assist (experimental), using the QnABot on AWS with [Large Language Model for Query Disambiguation, and Generative Question Answering](https://github.com/aws-solutions/qnabot-on-aws/blob/feature/llm-summarize/docs/LLM_Retrieval_and_generative_question_answering/README.md) - see [Agent Assist](./lca-agentassist-setup-stack/README.md#)
- New Agent Assist interaction widget on the LCA Call Detail page allows agents to directly query the Agent Assist Lex Bot / QnABot to execute Kendra knowledge base queries, FAQ lookups, or to automate actions during or after the call.
- [Amazon Chime SDK Call Analytics](https://aws.amazon.com/about-aws/whats-new/2023/03/amazon-chime-sdk-call-analytics/) supported (as an alternative to LCA's CallTranscriber Lambda).
  - _Known Issue: Post Call Analytics (PCA) integration is disabled when using Amazon Chime SDK Call Analytics - will be fixed in a later release._
- [Real-Time Voice Tone Analysis](https://aws.amazon.com/blogs/aws/amazon-chime-sdk-call-analytics-real-time-voice-tone-analysis-and-speaker-search/) supported when using Amazon Chime SDK Call Analytics. When selected during stack deployment, the LCA Call Detail UI now shows an additional chart reflecting the last 30sec rolling average of caller and agent voice tones.
- Saleforce/CRM integration plug-in - see [Boosting agent productivity with Salesforce integration for Live Call Analytics](./plugins/salesforce-integration/README.md)

### Changed

- Update Code Build version
- Improve throughput by decoupling agent assist inference latency from main call event processor - now runs asynchronously in parallel with transcript processing.
- Refactored call event processor code to normalize and merge handling of Contact Lens, Transcribe, Transcribe Call Analytics, and Amazon Chime SDK Call Analytics messages.

## [0.7.2] - 2023-05-03

### Fixed

- Hot fix for S3 bucket ACL changes
- Changed default function memory for Call Event Processor Lambda

## [0.7.1] - 2023-03-11

### Added

- Transcribe Call Analytics "POST_CALL" categories now displayed in the LCA UI after call ends when using Transcribe 'analytics' mode.
- New download buttons on call details page to save call summary or call transcript to local Excel file.
- Add configurable sentiment score thresholds for fine grain control over negative and positive sentiment scoring when using Transcribe 'standard' mode.
- Update Agent Assist QnAbot version to v5.3.0 with [optional semantic search](https://github.com/aws-solutions/qnabot-on-aws/blob/main/docs/semantic_matching_using_LLM_embeddings/README.md) support for FAQs.

### Changed

- Improved performance and scalability changes (tested to 300 concurrent calls):
  - Refactor call aggregate logic to make call event processor fully stateless - removed dependency on Kinesis tumbling window to enable multiple concurrent invocations per shard.
  - enable AppSync resolver caching to enable fast/efficient call state queries from call event processor.
  - enable Kinesis Enhanced Fan Out to reduce message read latency.
  - increase call event processor Lambda memory to 5120M to reduce start time and enable greater message handling concurrency in each invocation.
  - avoid multiple mutation retries on 'put item condition failure' - where retries will not resolve the condition and merely extend function duration.
  - call event processor now invokes transcript summarization asynchronously, avoiding blocking and choking execution concurrency limits.
  - retries for Start Transcription Stream if exceptions thrown, to allow improved tolerance for call bursts resulting in temporary TPS limit exceeded errors.
- Fix issue with START_CALL_PROCESSING event rule ( introduced in 0.7.0 with the multiple LCA stack fix).
- Fix issue introduced in 0.7.0 preventing invocation of custom transcript processing Lambda Hook function
- Miscellaneous improvements to test scripts - see [README](./lca-chimevc-stack/asterisk-test-scripts/README.md)
- Additional minor fixes - see commit history

## [0.7.0] - 2023-02-12

### Added

- Experimental generative transcript summarization to provide a short paragraph with a synopsis of each completed call; use the built-in summarization model or experiment with custom language models or APIs of your choice. See [Transcript Summarization](./lca-ai-stack/TranscriptSummarization.md).
- Utility Lambda function that retrieves call transcription from DynamoDB. See [Fetch Transcription Lambda](./lca-ai-stack/FetchTranscriptLambda.md).
- Optional translation of live or completed call transcripts into language of choice, using Amazon Translate.
- Ability to disable display of agent channel transcription in the call transcript pane.
- Test scripts for simulating phone calls. See [Asterisk Test Scripts](./lca-chimevc-stack/asterisk-test-scripts/README.md).
- LCA client utility to make it easier to test Call Event Processors and LCA UI without having to actually make a phone call. See [LCA Client](./utilities/lca-client/README.md).
- Download button on Calls page to save call list to local Excel file.
- Default audio recording used by demo Asterisk server now plays the agent side of the [Agent Assist demo script](./lca-agentassist-setup-stack/agent-assist-demo-script.md).

### Changed

- Fix bug in Call Transcriber Lambda that caused double transcription ifAmazon Chime SDK Voice Connector can't differentiate between caller and agent streams (caused by SBC not configured to use RFC 7865 metadata).
- Fixed issue where multiple LCA stacks in the same region processed calls from each other's Chime VS streams. Now each stack uses only it's own Chime VC instance, so you can run multiple stacks in the same account and region.
- Miscellaneous dependabot updates

## [0.6.0] - 2022-11-27

### Added

- Supports new [Amazon Transcribe Real-time Call Analytics](https://aws.amazon.com/transcribe/call-analytics/) streaming API
- Real-time Detected Issues, Call Categories, and Alerts
- Real-time Category and Alert Notifications via Amazon SNS subscriptions. See [Category Notifications](./lca-ai-stack/Notifications.md).
- Post call analytics (without additional transcription costs) through integration with the companion [Post Call Analytics (PCA)](https://www.amazon.com/post-call-analytics) solution.
- QnABot designer markdown answers enable rich text and media in Agent Assist messages

### Changed

- Latest QnABot (v5.2.4) now used for agent assist
- Improved logging in Call Transcriber lambda
- Extend payload for Agent Assist Lambda to include `dynamodb_table_name` and `dynamodb_pk`, so Lambdas can query the LCA Call Event table in DynamoDB to retrieve call metadata.

## [0.5.2] - 2022-10-20

### Added

- Support for using Transcribe Custom Language Models.
- Option to suppress partial transcription segments.
- Support for custom transcript processing logic via a user provided Lambda Hook function. See [Transcript Lambda Hook Function](./lca-ai-stack/TranscriptLambdaHookFunction.md).
- ChimeVC call initialization Lambda hook can attach optional arbitrary metadata json object to call record, for use by downstream custom applications via the LCA graphQL API or DynamoDB event sourcing table. See [ChimeVC call initialization Lambda hook](./lca-chimevc-stack/LambdaHookFunction.md).
- Ability to selectively disable call recordings using [ChimeVC call initialization Lambda hook](./lca-chimevc-stack/LambdaHookFunction.md).

### Fixed

- Agent Assist configuration is now correctly maintained during stack updates when CallEventProcessor function is replaced.

## [0.5.1] - 2022-09-30

### Added

- CallTranscriber now has the ability to delay start of call processing until a (new) START_CALL_PROCESSING event is received - useful when call is streaming is initiated when call is established but transcribing should be delayed until after IVR navigation and hold time and triggered only when agent and caller are connected. See [Start call processing for a in-progress call](./lca-chimevc-stack/StartCallProcessingEvent.md).
- Support markdown and html rendering in LCA UI
- Use QnABot markdown response when available, to take advantage of new markdown rendering feature for Agent Assist.

### Changed

- CallTranscriber Lambda now has it's own DynamoDB table and no longer shares a table with the AISTACK components.

### Fixed

- Add ExpiresAfter fields to AGENT_ASSIST messages, used as TTL in DynamoDB
- Check StackName length and fail fast if length would cause downstream stack failures caused by long resource names in nested stacks. Max Stack Name is now 25 characters.
- Provide 'reason' message for stack custom resource failures.
- Expand Troubleshooting README.

## [0.5.0] - 2022-09-11

### Added

- AgentID call attribute and associated API support for setting, displaying, sorting, and filtering calls by Agent. See [Setting AgentId](./lca-chimevc-stack/SettingAgentId.md).
- AgentId field automatically assigned from Amazon Connect contact events when using `Amazon Connect Contact Lens` as the Call Audio Source.
- Support for custom logic via a user provided Lambda function to selectively choose which calls to process, toggle agent/caller streams, assign AgentId to call, and/or modify values for CallId and displayed phone numbers. See [Lambda Hook Function for SIPREC Call Initialization](./lca-chimevc-stack/LambdaHookFunction.md).
- Configurable retention period for call records (default 90 days). Records and transcripts that are older than this number of days are permanently deleted.
- UI supports new 'Load: 2 hrs' option for improved performance in high volume contact centers.

### Changed

- Moved transcriber Lambda out of AI stack and into ChimeVC stack.
- Remove code for call event stream processing lambda no longer used since LCA v0.4.0.
- Rename TranscriptProcessorLambda to CallEventProcessorLambda to reflect that it will process call analytics and contact metadata events in addition to transcripts.
- Rename lca-ai-stack CF template.
- Asterisk demo server reinstalled on instance reboot such as during stack updates containing Asterisk configuration or version changes.
- Asterisk demo installation script is no longer dependent on hardcoded Asterisk version.
- Asterisk demo server is reloaded each hour to resolve observed busy tones in previous releases.
- Default Asterisk demo server version is now v19.
- DynamoDB event sourcing table now maintains only one item per transcript segment and no longer maintains partial segments.
- ChimeVC CallTranscriber Lambda function now emits one Call START event when both call streams are ready (as opposed to one per call stream), eliminating 'item already exists' errors in the CallEventProcessorLambda lambda
- ChimeVC CallTranscriber Lambda function memory footprint reduced to 768MB to improve cost efficiency with minimal latency tradeoff.
- README updates

## [0.4.1] - 2022-08-22

### Changed

- Remove E.164 type enforcement on CustomerPhoneNumber and SystemPhoneNumber. Any string value is now allowed, enabling calls to be processed when either/both CustomerPhoneNumber and SystemPhoneNumber fields are non E.164 strings.

## [0.4.0] - 2022-07-15

### Added

- Introducing support for real time Agent Assist features - see [Agent Assist README](lca-agentassist-setup-stack/README.md).
- Added support for Amazon Connect Contact Lens as an optional call source - see [Amazon Connect Integration README](/lca-connect-integration-stack/README.md)

### Changed

- Latest Asterisk version (18) for demo PBX
- Solution title is now "Live Call Analytics with Agent Assist"

## [0.3.0] - 2022-05-18

### Added

- Support for [Genesys AudioHook](https://help.mypurecloud.com/articles/audiohook-integration-overview/). You can now
  stream call audio into the Live Call Analytics solution from a Genesys AudioHook. See the details in the
  [README](https://github.com/aws-samples/amazon-transcribe-live-call-analytics/blob/main/lca-genesys-audiohook-stack/README.md)
  file

### Changed

- Changed the audio stream consumer and transcriber component to run on AWS Lambda instead of AWS Fargate. This new Call
  Transcriber provides the following benefits:
  - Reduced Amazon Transcribe processing cost. The agent and caller audio streams are now automatically merged into a
    single stereo stream. Both the agent and caller audio streams are transcribed in a single session
  - Longer call duration handling. The Fargate consumer had a timeout of 42 minutes. The new Lambda based transcriber
    has a mechanism that allows to transparently handoff the call processing from one Lambda invocation to another
  - Faster scaling on sudden call volume spikes
  - Simplified stereo recording process. The stereo recordings are now produced by the Call Transcriber instead of
    merging recording files after the call was done
  - Serverless!
- Additional language support
- The Call Event Stream Processor Lambda function now runs on the arm64 architecture. The new Call Transcriber Lambda
  function also runs on arm64. The build was updated to enable arm64 emulation when using the SAM CLI in Amazon Linux 2
- Partial transcript events are now only persisted for 1 day in DynamoDB. Final transcript events are persisted for 90
  days by default
- Separated the build and development python virtual environments to avoid development dependencies interfere with the
  SAM CLI
- Updated the Python GraphQL client in the Call Event Stream Processor Lambda function to the released stable version
  [gql v3.2.0](https://github.com/graphql-python/gql/releases/tag/v3.2.0). This version is now out of pre-release which
  removed the need for the Makefile based SAM CLI build of the Lambda layer. The library also added direct support for
  AppSync which removed the need for the custom AppSync transport and authentication code. The Makefile build and custom
  AppSync code has been deleted
- Updated dependency versions of various components including:
  - Web UI
  - Call Event Processor Lambda function
  - Project build and development
- Updated nodejs and npm versions used in CodeBuild to build the UI
- Fixed demo agent recording download issue

### Removed

- The resources associated with the Fargate consumer such as Fargate cluster/service/task definition, VPC, SQS queues
  and autoscaling were removed in favor of the new Lambda based Call Transcriber
- The resources associated with the creation of post call stereo recordings (including Lambda functions, S3 bucket and
  SQS queue) were removed in favor of the new Call Transcriber that merges the audio into stereo recordings at call time

## [0.2.1] - 2022-02-02

### Fixed

- Fixed semantic version in main cloudformation template

## [0.2.0] - 2022-02-02

### Added

- Added script to update semantic versions in source files. `scripts/update-version.sh`
- Added TROUBLESHOOTING.md for instructions on how to check for errors
- Added web UI admin user creation via CloudFormation. The email address of the admin user is
  passed via the `AdminEmail` CloudFormation parameter. An initial temporary password is
  automatically sent to this user via email. This email also includes the link to the web UI
- Added CloudFormation parameter to enable or disable Sentiment Analysis. See the
  `IsSentimentAnalysisEnabled` parameter
- Added CloudFormation mapping to configure the Amazon Comprehend language from the selected Amazon
  Transcribe language
- Added support to select the Spanish language (es-US) in the CloudFormation template. The
  CloudFormation template now allows to select either English (en-US) or Spanish (es-US) using the
  `TranscribeLanguageCode` parameter. **NOTE:** Content redaction is only available when using the
  English language (en-US). It is automatically disabled when using other languages
- Added the `CloudFrontAllowedGeos` CloudFormation parameter to control the CloudFront geographic
  restrictions. You can specify a comma separated list of two letter country codes (uppercase ISO
  3166-1) that are allowed to access the web user interface via CloudFront. For example: US,CA.
  Leave empty if you do not want geo restrictions to be applied

### Changed

- The CloudFormation `AllowedSignUpEmailDomain` parameter is now optional. If left empty, signup
  via the web UI is disabled and users will have to be created using Cognito. If you configure a
  domain, **only** email addresses from that domain will be allowed to signup and signin via the
  web UI
- The CloudFront distribution now defaults to no geographic restrictions. There's a new parameter
  named `CloudFrontAllowedGeos` that allows you to add geographic restrictions. If you leave this
  parameter empty, the previous geographic restriction will be removed on an update to this version.
  The previous version had a hardcoded value that set the restriction to `US` only. Set the
  `CloudFrontAllowedGeos` to `US` if you want to preserve the previous default configuration after
  updating to this version

### Fixed

- Reverted kvs stream parser library version workaround
- Asterisk server will wait for voice connector to get a phone number before initializing

## [0.1.0] - 2021-12-16

### Added

- Initial release

[Unreleased]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.8.3...develop
[0.8.3]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.8.2...v0.8.3
[0.8.2]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.8.1...v0.8.2
[0.8.1]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.8.0...v0.8.1
[0.8.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.7.2...v0.8.0
[0.7.2]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.7.1...v0.7.2
[0.7.1]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.7.0...v0.7.1
[0.7.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.6.0...v0.7.0
[0.6.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.5.2...v0.6.0
[0.5.2]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.5.1...v0.5.2
[0.5.1]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.5.0...v0.5.1
[0.5.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.4.1...v0.5.0
[0.4.1]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.4.0...v0.4.1
[0.4.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.2.1...v0.3.0
[0.2.1]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.2.0...v0.2.1
[0.2.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/aws-samples/amazon-transcribe-live-call-analytics/releases/tag/v0.1.0
