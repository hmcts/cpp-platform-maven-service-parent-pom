# Change Log

All notable changes to this project will be documented in this file, which follows the guidelines
on [Keep a CHANGELOG](http://keepachangelog.com/).

This project adheres to [Semantic Versioning](http://semver.org/).

## [Unreleased]

# [17.103.8-M1] - 2025-11-20
### Changed
- Used JsonFactory instead of Json.create methods
- Refactor JsonObject usages to more proper api
- Fix HttpClient lifecycle.

# [17.103.5] - 2025-11-06
### Changed
- Update event-store to 17.103.3-M1 for:
  - ReplaySingleEvent JMX commands can now take an optional commandRuntimeString of the event source name
    to work with MI contexts

# [17.103.3] - 2025-09-11
### Change
- Update cpp-platform-libraries to 17.103.3

# [17.103.2] - 2025-08-28
### Change
- Update cpp-platform-libraries to 17.103.2 to include CCT-1530 changes


# [17.103.1] - 2025-08-18
- Update event-store 17.103.1 for:
  - Refactoring of error handling to make database access more efficient. These include:
    - Delete of stream errors sql no longer cascades to delete any orphaned hashes. Instead, orphaned hashes are found and deleted if necessary
    - Locking of stream_status table when publishing events, no longer calls error tables updates on locking errors
    - Streams no longer marked as fixed by default and will only mark as fixed if stream previously broken
    - We now check that any new error not a repeat of a previous error before updating stream_error tables.
      - If the error is different, we now lock stream_status and updates error details.
      - If the error is same, `markSameErrorHappened(...)` only updates stream_status.updated_at.
    - We now execute the lock of steam_status table before calculating stream statistics

# [17.103.0] - 2025-07-16
### Added
- Add micrometer metrics
- Micrometer metrics are now bootstrapped in cpp-platform-libraries at application startup using `MicrometerMetricsWildflyExtension`
### Changed
- Release JMX command execution to work with selfhealing feature (via event-store)
- Release file-service extraction changes (via framework-libraries)
- Make framework REST endpoints available to contexts
- Azure metrics registered with global tags and factories
- Add framework rest endpoint to fetch streams by errorHash
- Azure metrics registry is no longer bootstrapped if metrics disabled
- Update cpp.common-bom to Include netty-bom instead of individual netty library versions.
- DD-38618 DLRM changes related to unified search (cpp-platform-libraries)
- Update to framework `D` `17.103.x`:
  - Insert into stream_buffer table during event publishing is now idempotent
  - Run each event sent to the event listeners in its own transaction
  - Update the `stream_status` table with `latest_known_position`
  - Mark stream as 'up_to_date' when all events from event-buffer successfully processed
  - New column `latest_known_position` in `stream_status table`
  - New column `is_up_to_date` in `stream_status table`
  - New liquibase scripts to update stream_status table
  - New SubscriptionManager class `NewSubscriptionManager`to handle the new way of processing events
  - New replacement StreamStatusRepository class for data access of stream_status table
  - Change name of jndi value for self-healing from `event.error.handling.enabled` to `event.stream.self.healing.enabled`
  - New REST endpoint to fetch list of active errors
  - New REST endpoint to fetch stream errors by streamId and errorId
  - Fetch streams by streamId and hasError
  - Extended RestPoller to include custom PollInterval implementation and introduced FibonacciPollWithStartAndMax class

## [17.102.6] - 2025-04-28
### Changed
- Update cpp.common-bom to Include netty-bom instead of individual netty library versions.

## [17.102.5] - 2025-04-17
### Changed
- Update framework to 17.102.2 for:
    - Extended RestPoller to include custom PollInterval implementation and introduced FibonacciPollWithStartAndMax class
- DD-38618 DLRM changes related to unified search (cpp-platform-libraries)

## [17.102.4] - 2025-03-18
### Changed
- Update event-store to 17.102.2 for:
    - Removal of error handling from BackwardsCompatibleSubscriptionManager
- Automatically sync up cpp.platform-libraries.version property value with pom version (in cpp-platform-libraries)

## [17.102.3] - 2025-03-18
### Changed
- Update event-store to 17.102.1 for:
    - - Oversized messages are now logged as `WARN` rather than `ERROR`
- Update cpp-platform-maven-common-bom to 17.102.1

## [17.102.2] - 2025-03-10
### Changed
- Update event-store to 17.102.0-M7 for:
    - Stream error handling now uses same database connection for all error handling database updates
    - Failing retries of a previously stored error no longer update the error tables and only the first failure of that event is recorded
    - Renamed `component_name` column in `stream_error` to `component`
    - Streams no longer marked as fixed if events remain stuck in the stream-buffer
    - Added new `updated_at` column to `stream_status` table

## [17.102.1] - 2025-02-28
### Changed
- Update event-store to 17.102.0-M5 for:
    - The columns `stream_id`, `component_name` and `source` on the `stream_error` table are now unique when combined
    - Inserts into `stream_error` now `DO NOTHING` if a row with the same `stream_id`, `component_name` and `source` on the `stream`error` already exists
    - Inserts into `stream_error` are therefore idempotent
    - No longer removing stream_errors before inserting a new error, as the insert is now idempotent

## [17.102.0] - 2025-02-27
### Added
- Error handling for event streams:
    - New table `stream_error` in viewstore
    - Exceptions thrown during event processing now stored in stream_error table
    - New nullable column `stream_error_id` in stream status table with constraint on stream_error table
    - New nullable column `stream_error_position` in stream status table
    - Exception stacktraces are parsed to find entries into our code and stored in stream_error table
    - New Interceptor `EntityManagerFlushInterceptor` for EVENT_LISTENER component that will always flush the Hibernate EntityManager to commit viewstore changes to the database
    - New JNDI value `event.error.handling.enabled` with default value of `false` to enable/disable error handling for events
### Changed
- Bump version to 17.102.x
- Optimised SnapshotJdbcRepository queries to fetch only required data

## [17.101.1] - 2025-04-14
### Changed
-- updated to include sourceSystemReference into unified search part of DLRM

## [17.101.0] - 2025-01-29
### Changed
- Bump platform-libraries-parent-pom to 17.101.3
- Bump framework-libraries to 17.101.2
- Bump microservice-framework to 17.101.6
- Bump event-store to 17.101.5
- Bump progression version to 17.0.52
- Optimised SnapshotJdbcRepository queries to fetch only required data
- Update postgresql.driver.version to 42.3.2 (through maven-parent-pom)
- Update common-bom to 17.101.0
### Added
- Expose prometheus metrics through /internal/metrics/prometheus endpoint
- Provide timerRegistrar bean to register timer with metricsRegistry
- Save aggregate snapshots asynchronously in the background when we have a large amount of event on a single stream. Default it 50000. This is configurable via JNDI var snapshot.background.saving.threshold
- Add 'liquibase.analytics.enabled: false' to all liquibase.properties files to stop liquibase collecting anonymous analytics if we should ever upgrade to liquibase 4.30.0 or greater. Details can be found here: https://www.liquibase.com/blog/product-update-liquibase-now-collects-anonymous-usage-analytics
- Add dependency for org.ow2.asm version 9.3 (through maven-common-bom)
### Security
- Update com.jayway.json-path to version 2.9.0 to fix **security vulnerability CWE-787**
  Detail: https://cwe.mitre.org/data/definitions/787.html (through maven-common-bom)
- Update commons.io to 2.18.0 to fix security vulnerability CVE-2024-47554
  Detail: https://nvd.nist.gov/vuln/detail/CVE-2024-47554 and https://cwe.mitre.org/data/definitions/400.html
### Removed
- Removed OWASP cross-site scripting check on html rest parameters introduced in microservice-framework release 17.6.1 and service-parent-pom 17.100.3

## [17.100.3.1] - 2024-12-19
- Disable service-parent-pom and core-domain enforcer rules

## [17.100.3] - 2024-12-16
### Changed
- Post Java 17 release of the framework
- Update framework to 17.100.0 for the changes:
  - Improved the fetching of jobs by priority from the jobstore by retrying with a different priority if the first select returns no jobs
  - Refactor of File Store
  - Re-introduce 'soft' delete of files in file store. 'deleted' files now have a 'deleted_at' column.
  - Adding maxSession for all MessageDrivenBeans
  - All events pulled from the event queue by the message driven bean now
    check the size of the message, and will log an error if the number of bytes
    is greater than a new jndi value `messaging.jms.oversize.message.threshold.bytes`
  - All rest http parameters in the generated rest endpoints are now encoded using owasp to
    protect against cross site scripting
  - Jmx MBean `SystemCommanderMBean` now only takes basic Java Objects to keep the JMX handling interoperable
  - Improved error messages printed whilst running framework-command-client.jar
  - Integrate metrics with prometheus meter registry
  - Provide REST endpoint for prometheus to scrape metrics (/internal/metrics/prometheus)
  - Split filestore `content` tables back into two tables of `metadata` and `content` to allow for backwards compatibility with liquibase
  - All JmxCommandHandlers must now have `commandName` String, `commandId` UUID and `JmxCommandRuntimeParameters` in their method signatures
  - Jmx commands can now have and extra optional String `command-runtime-string` that can ba
    passed to JmxCommandHandlers via the JmxCommandHandling framework
  - Release audit client async mode changes (via cpp.platform.libraries)
### Added
- New method 'payloadIsNull()' on DefaultJsonEnvelope, to check if the payload is `JsonValue.NULL` or `null`
- New column `buffered_at` on the stream_buffer tables to allow for monitoring of stuck stream_buffer events
- New Jmx command `RebuildSnapshotCommand` and handler that can force hydration and generation of an Aggregate snapshot
- New parameter 'JmxCommandRuntimeParameters' to JMX commands
- Add hybrid audit client which writes audit event payload to both artemis topic and azure datalake storage (via cpp.platform.libraries)
### Removed
- Removed `JmxCommandParameters` and `CommandRunMode` from JMX SystemCommanderMBean call
- Removed @MXBean annotation from Jmx interface class to change from MXBean to MBean
- Removed test classes erroneously included in framework-command-client.jar
- Remove debug logging of request payload in HybridAuditClient (via cpp-platform-libraries)
### Fixed
- Fix where null payloads of JsonEnvelopes get converted to `JsonValue.NULL` and cause a ClassCastException
- All JsonEnvelopes that have null payloads will now:
  - return `JsonValue.NULL` if `getPayload()` is called
  - throw `IncompatibleJsonPayloadTypeException` if `getPayloadAsJsonObject()` is called
  - throw `IncompatibleJsonPayloadTypeException` if `getPayloadAsJsonArray()` is called
  - throw `IncompatibleJsonPayloadTypeException` if `getPayloadAsJsonString()` is called
  - throw `IncompatibleJsonPayloadTypeException` if `getPayloadAsJsonNumber()` is called
- Fixed error in RegenerateAggregateSnapshotBean where a closed java Stream was reused
- JdbcResultSetStreamer now correctly streams data using statement.setFetchSize(). The Default fetch size is 200. This can be overridden with JNDI prop jdbc.statement.fetchSize

## [17.100.0] - 2024-10-22
### Changed
- service parent pom enforcer rule now checks for latest major and minor versions and ignores patch version (via cpp.platform.maven.parent-pom)

## [17.21.21] - 2024-09-05
### Changed
- PEG-392: eventProcessor destinationType (queue|topic) to come from event-sources.yaml
- Update cpp.platform.libraries in order to:
  - Fix problem of guava-testlib on compile scope in cpp-platform-data-utils-rest:

## [17.21.21] - 2024-08-02
### Changed
- Provide capability to configure jgitflow develop branch name through 'jgitflow.maven.developBranchName' property (via cpp.platform.maven.parent-pom)


## [17.21.19] - 2024-07-24
### Fixed
- Jackson single argument constructor pojo issue

## [17.21.18] - 2024-07-17
### Fixed
- Downgrade azure-storage-file-datalake to 12.10.1 to stop incompatible netty jars getting included in wars

## [17.21.17] - 2024-07-16
### Added
- Add enforcer rules to verify core domain and service parent pom are on latest release version (via cpp.platform.maven.parent-pom)
- FRAMEWORK_LIBRARIES_VERSION=17.5.2   FRAMEWORK_VERSION=17.5.2    EVENT_STORE_VERSION=17.5.2
  
## [17.21.16] - 2024-07-16
### Changed
- Updated common-bom to 17.21.8 in order to:
  - Update all azure libraries to latest versions

## [17.21.15] - 2024-06-20
- Remove duplicate jgitflow maven plugin (through cpp.platform.maven.parent-pom)

## [17.21.14] - 2024-06-20
- Add sonar-maven-plugin plugin (through cpp.platform.maven.parent-pom)

## [17.21.13] - 2024-06-17
### Added
- Add jgitflow maven plugin (through cpp.platform.maven.parent-pom)
- Enhance JmsMessageConsumerClient to retrieve message as JsonObject (through microservice-framework)
- Add docmosis dependency to dependency management (through cpp.platform.common-bom)

## [17.21.12] - 2024-06-12
### Added
- Add maven-sonar-plugin to pluginManagement (through maven-parent-pom)

## [17.21.11] - 2024-06-06
### Changed
- Update framework to 17.5.0 for:
  - JmsMessageConsumerClientProvider now returns JmsMessageConsumerClient interface rather than the implemening class

## [17.21.10] - 2024-06-04
### Changed
- Add method to SystemCommanderMBean interface to invoke system command without supplying CommandRunMode (through micro-service-framework changes)

## [17.21.9] - 2024-06-03
### Changed
- Break dependency on framework-command-client in test-utils-jmx library (by micro-service-framework changes)

## [17.21.8] - 2024-05-29
### Added
- Add REPLAY_EVENT_TO_EVENT_LISTENER and REPLAY_EVENT_TO_EVENT_INDEXER system command handlers to replay single event (through micro-service-framework/event-store)
- Add LOG_RUNTIME_ID system command and it's handler to log command runtime id (through micro-service-framework)
### Changed
- Renamed JmsMessageProducerClientBuilder to JmsMessageProducerClientProvider (through micro-service-framework)
- Renamed JmsMessageConsumerClientBuilder to JmsMessageConsumerClientProvider (through micro-service-framework)

## [17.21.7] - 2024-05-15
### Changed
- Release microservice-framework changes for adding junit CloseableResource that manages closing jms resources

## [17.21.6] - 2024-05-13
### Added
- Release micro-service-framework changes for adding jms message clients for effective management of jms resources in integration tests

## [17.21.5] - 2024-02-08
### Changed
- Release jobstore retries migration liquibase script via framework-libraries

## [17.21.3] - 2023-12-13
### Changed
- Release jobstore retry capability (through framework-libraries)

## [17.21.3] - 2023-11-29
### Changed
- Update common-bom to 17.21.3

## [17.21.2] - 2023-11-28
### Changed
- Update cpp.common-bom to version 17.21.2

## [17.21.1] - 2023-11-10
### Changed
Release cpp maven common bom (Remove transformation related dependencies)

## [17.21.0] - 2023-11-09
### Changed
- Moved all generic library dependencies and versions into GitHub common-bom
- Centralise all generic library dependencies and versions into maven-common-bom
- Centralise dependencies after consolidating contexts data
- Moved hard coded dependency on drools-mvel to common-bom
### Security
- Update org.json to version 20231013 to fix **security vulnerability CVE-2023-5072**
  Detail: https://nvd.nist.gov/vuln/detail/CVE-2023-5072
- Update plexus-codehaus to version 3.0.24 to fix **security vulnerability CVE-2022-4244**
  Detail: https://nvd.nist.gov/vuln/detail/CVE-2022-4244
- Update apache-tika to version 1.28.3 to fix **security vulnerability CVE-2022-30973**
  Detail: https://nvd.nist.gov/vuln/detail/CVE-2022-30973
- Update google-guava to version 32.0.0-jre to fix **security vulnerability CVE-2023-2976**
  Detail: https://nvd.nist.gov/vuln/detail/CVE-2023-2976

## [17.10.5] - 2023-09-06
### Removed
- Update framework to 17.1.1 in order to
  - Remove `clientId` from the header of all generated Message Driven Beans
### Fixed
- Update event-store to 17.1.2 in order to
  - Fix IndexOutOfBoundsException in ProcessedEventStreamSpliterator during catchup
- 
## [17.10.4] - 2023-08-02
- Merge release/framework-8.x.x to bring 10.9.2, 10.9.3, 10.9.4 changes

## [10.9.4] - 2023-06-08
### Changed
- Updated event-store to 8.3.3 in order to
  - Limit logging of MissingEventRanges logged to sensible maximum number.
  - Add new JNDI value `catchup.max.number.of.missing.event.ranges.to.log`

## [10.9.3] - 2023-06-06
### Fixed
- Updated event-store to 8.3.2 in order to
  - Fix Logging of missing event ranges to only log on debug

## [10.9.2] - 2023-06-02
### Changed
- Update event-store to 8.3.1 in order to:
- Fix batch fetch of processed events to load batches of 'n' events
  into a Java List in memory

## [17.10.3] - 2023-08-02
### Changed
- Update to junit5 and update surefire, failsafe plugins
- Update wiremock-test-utils version

## [17.0.3] - 2023-07-06
### Changed
Fix jacoco coverage issue (by releasing 17.0.1 parent-pom)

## [17.0.2] - 2023-06-16
### Added
- Framework plugin dependencies to fix maven warnings/errors
### Changed
- Moved various third party library version numbers to common-bom
### Security
- Update org.json to version 20230227 to fix **security vulnerability CVE-2022-45688**
  Detail: https://nvd.nist.gov/vuln/detail/CVE-2022-45688

## [17.0.1] - 2023-06-05
### Changed
- Updated library-parent-pom to 17.0.3 in order to bring wiremock regex fix
- Upgrade wiremock service to latest version 17.0.0

## [17.0.0] - 2023-05-24
### Changed
- Update to Java 17
- Bump the version to 17.x.x to indicate formal framework 17 release
- Update to JEE 8.0.1
- Update to wildfly 26.1.2
- Update rest-client libraries for wildfly 26.1.2
- Filestore now does hard delete in the database rather than just marking as deleted
- Update plugins.jaxb2.version to 0.15.2
- Fix pojo generators by making additionalProperties assignment null safe (through
  framework-libraries)
- MessageProducerClient and MessageConsumerClient are now idempotent when start is called
- A default name of `jms.queue.DLQ` rather than the original name of `DLQ`
- A new constructor to pass the name in if you don't want the default name
- New builder `MessageConsumerClientBuilder` that allows ActiveMQ connection parameters to be
  specified
- Update library-parent-pom to bring mockito upgrade changes and commons.text.version maven property
  fix
- Update jackson version to 2.12.7
- Update library-parent-pom/cpp.platform.libraries version to bring unifiedsearch-healthchecks
  dependency group id fix
- Close the client in DefaultRestClientProcessor
- Extract unifiedsearch and workmanagement healthcheck modules to healthcheck-parent module
- Update jboss-logging version to 3.5.0.Final
- Update slf4j/log4j bridge jar from slf4j-log4j12 to slf4j-reload4j
- Update artemis-jms-client to 2.10.1
- Remove strict checking of liquibase.properties files
- Update liquibase to 4.10.0
- Update jayway libraries
- Updated Jackson version to 2.10.5
### Added
- Add new healthcheck apis
- Add new healthcheck for elastic search
- Add common healthcheck for camunda
- Add common dependencies used by Deltaspike tests
### Removed
- Remove log4j-over-slf4j as it is now replaced by slf4j-reload4j
- Remove illegal-access flag from surefire plugin (through framework maven-parent-pom)
- Remove obsolete authorisation-service as it is now replaced by feature flags
- Remove unnecessary logging of 'skipping generation' message in pojo generator
- Remove the use of the trigger on event_log table as part of event publishing
- Remove framework-generator dependencies in wars
### Security
- Updated log4j2 to 2.15.0 to fix security
  vulnerability https://www.randori.com/blog/cve-2021-44228/
- Update hibernate version to 5.4.24.Final
- Update jackson.databind version to 2.12.7.1
- Update log4j to 2.17.2. Log4j 1 no longer used

## [10.9.0] - 2023-05-19
### Changed
- Update event-store to 8.3.0 in order to:
  - Fetch of processed events from viewstore during catchup is now batched, to stop transaction
    timeout when there are millions of events
  - Add new JNDI value `catchup.fetch.processed.event.batch.size` that configures the size of the
    batch. Default value is 100,000

## [10.8.17] - 2023-01-31
### Changed
- Updated event-store version to 8.2.2 in order to:
  - Remove unnecessary indexes from event_log table
  - Updated platform-libraries version to 8.0.16 for Word Document Generation for DD-25099

## [10.8.15] - 2023-01-04

### Changed
- Updated platform-libraries version to 8.0.15 for SNI-2020

## [10.8.13]- 2022-07-25
### Changed
- Updated framework-libraries to 8.0.5 in order to
  - Update API path for alfresco read material endpoint

## [10.8.12]- 2022-07-20
### Changed
- Updated platform-libraries version to 8.0.5-cct1139-SNAPSHOT to add hasSharedResults search index
  for unified search

## [10.8.12]- 2021-10-13
### Changed
- Updated platform-libraries version to 8.0.5-cct1139-SNAPSHOT to add hasSharedResults search index
  for unified search

## [10.8.11] - 2022-05-28
### Changed
- Updated platform-libraries version to 8.0.12 for SnL HTTP DELETE with payload

## [10.8.10] - 2022-05-06
### Changed
- Updated platform-libraries version to 8.0.11
  - changed the document generator client in library to point to system doc generator as part of
    CCT-1118

## [10.8.8] - 2022-03-23
### Changed
- Updated platform-libraries version to 8.0.9
  - Reverted changes implemented for document generator client in library to point to system doc
    generator as part of CCT-1118

## [10.8.6] - 2022-03-18
### Changed
- Updated platform-libraries version to 8.0.7
  - changed the document generator client in library to point to system doc generator as part of
    CCT-1118

## [10.8.5] - 2022-03-11
### Changed
- Update to framework 8.0.4:
  - Filestore no longer supports in memory database. Postgres only

## [10.8.4] - 2022-03-08
### Changed
- Update to framework 8.0.3 to enable:
  - Filestore now does hard delete when deleting files rather than just marking as deleted

## [10.8.3] - 2022-02-17
### Changed
- Update to framework 8.x.x to:
  - Remove trigger from event_log table
  - Fix possible out of memory exception when running catchup for more than 4,000,000 events
  - Update cpp.platform.libraries to 8.0.4
  - Bumped version to reflect framework 8 version

## [10.5.0] - 2021-12-24
### Changed
- ElasticSearch client upgrade to 7.16.2

## [10.4.0] - 2021-12-14
### Changed
- Updated log4j2 to 2.15.0 to fix security
  vulnerability https://www.randori.com/blog/cve-2021-44228/
- Downgrade to framework 7.x.x for emergency fix

## [10.5.0] - 2021-12-24
### Changed
- ElasticSearch client upgrade to 7.16.2


## [10.8.2] - 2021-10-13
### Changed
- Downgrade liquibase to 3.5.3 to match the version in the framework

## [10.8.1] - 2021-10-08
### Changed
- DD-12921 - Added bulk system ID mapper client api

## [10.8.0] - 2021-10-05
### Changed
- Update to framework 8.0.0 as the final Java 8 version of the framework
- Bump version to 10.8.0 to match framework 8 version
- Update service-parent-pom to 8.0.0
- Update to framework 8.0.0 in order to
  - Add the fix for out of memory errors when running catchup on very large datasets

## [10.3.10] - 2021-08-23
### Changed
- Fixing DD-15055 fixed query audit provider for large message

## [10.3.9] - 2021-07-08
### Changed
- Fixing DD-12772 Feature Toggle - Get features from Azure throws error when fails

## [10.3.8] - 2021-04-30
### Changed
- Updated platform-libraries version to 7.2.8 to add cps search index for unified search

## [10.3.7] - 2021-04-21
### Changed
- Updated platform-libraries version to 7.2.7 to bring in handling of http 400 bad request in
  RestClientProcessor

## [10.3.6] - 2021-02-18
### Changed
- Updated platform-libraries version to 7.2.5

## [10.3.4] - 2021-02-18
### Changed
- Updated platform-libraries version to 7.2.4 for new remap function in system-id-mapper client

## [10.3.3] - 2021-02-15
### Changed
- Updated platform-libraries version to 7.2.3 for jndi variables for audit enabling/disabling at app
  level

## [10.3.2] - 2020-12-08
### Changed
- Updated platform-libraries version to 7.2.2 to bring in feature toggling integration test stubbing
  utilities

## [10.3.1] - 2020-12-08
### Changed
- Updated framework-libraries to 7.2.1 to bring in feature toggling changes

## [10.2.2] - 2020-10-16
### Changed
- Updated framework-libraries to 7.1.5
  - Builders of generated pojos now have a `withValuesFrom(...)` method to allow the builder to be
    initialised with the values of another pojo instance
- Updated cpp.platform.maven-common-bom to 7.1.2
  - Added junit4-dataprovider version 2.6 to dependency management at test scope

## [10.2.0] - 2020-10-15
### Changed

- Updated common-bom to 7.1.0 to for various security fixes to libraries
  - junit:                4.12 ->   4.13.1      https://github.com/advisories/GHSA-269g-pwp5-87pp
  - commons.beanutils:    1.9.2 ->   1.9.4       https://github.com/advisories/GHSA-6phf-73q6-gh87
  - commons.guava:        19.0 ->   29.0-jre    https://github.com/advisories/GHSA-mvr2-9pj6-7w5j

## [10.1.2] - 2020-10-05

### Changed

- Updated common-bom to 7.0.5 to pull in the correct version of h2 (version 1.4.196)

## [10.1.0] - 2020-10-02

### Changed

- Framework libraries now stored in cloudsmith rather than bintray
- Artifactory now pulling framework libraries from cloudsmith
- Update framework-libraries version to 7.1.1
- Update platform-libraries version to 7.1.1
- Update event-store version to 7.1.1
- Update cpp.platform.libraries to 7.0.24

## [10.0.11] - 2020-09-11

### Changed

- Update framework-libraries version to 7.0.11
- Update platform-libraries version to 7.0.22
- Constructors of JsonObjectConverter classes updated to ease testing.
  see https://tools.hmcts.net/confluence/display/CTP/Changes+to+constructors+of+JsonObject+converter+classes

## [10.0.10] - 2020-09-01

### Changed

- Update event-store version to 7.0.9
- Update platform-libraries version to 7.0.21

## [10.0.9] - 2020-08-14

### Changed

- Update cpp.framework version to 7.0.10
- Update event-store version to 7.0.8
- Update platform-libraries version to 7.0.18

## [10.0.8] - 2020-08-13

### Changed

- Update framework-libraries version to 7.0.10
- Update cpp.framework version to 7.0.9
- Update event-store version to 7.0.7
- Update platform-libraries version to 7.0.17

## [10.0.7] - 2020-07-29

### Changed

-Updated platform-libraries.version to 7.0.15 (MVP fixes and fixed test-util bom dependency)

## [10.0.6] - 2020-07-29

### Changed

-Updated platform-libraries.version to 7.0.13 (Added final version of Aggregate serialization test
util)

## [10.0.5] - 2020-07-26
### Changed
-Updated platform-libraries.version to 7.0.12 (correction of service components in
platform-libraries-bom)

## [10.0.4] - 2020-07-25
### Changed
- Removed service-components dependencies, now in cpp.context.libraries/platform-libraries-bom
- Updated platform-libraries.version to 7.0.11
- Update cpp.framework to version 7.0.8
- Update event-store to version to 7.0.6
- Update common-bom to version to 7.0.3
- Update framework-libraries.version to 7.0.9

## [10.0.3] - 2020-07-08
### Changed
- Update cpp.platform.libraries to 7.0.5
  - This adds jndi variables for enabling/disabling auditing of context endpoints
- Update framework-libraries to version 7.0.9
- Update cpp.framework to version 7.0.7
- Update event-store to version 7.0.5
- Update common-bom to version 7.0.1

## [10.0.2] - 2020-06-22
### Changed
- Update cpp.platform.libraries to 7.0.3
  - This adds jndi variables for enabling/disabling auditing of context endpoints

## [10.0.1] - 2020-06-16
### Changed
- Moved service-components to cpp.platform.libraries
- Squashed all service-parent-poms into one parent pom file

## [10.0.0] - 2020-06-15
### Changed
- Bumped version to 10.0.0 to match update to framework 7.x.x world
- Upgrade framework-libraries to 7.0.8
- Upgrade cpp.framework to 7.0.6
- Upgrade event-store to 7.0.4

### Added
- Dependency cpp.platform-libraries 7.0.0 which is the new common project of all internal libraries

## [9.4.30] - 2020-05-18
### Changed
Upgrade unified-search-library to 2.0.4

## [9.4.29] - 2020-05-04
### Changed
- Upgrade cpp.context.access-control to 6.4.12

## [9.4.28] - 2020-04-24
### Changed

Upgrade framework-api to 4.3.0 Upgrade framework to 6.4.2 Upgrade event-store to 2.4.13 Upgrade
framework-generators to 2.4.4 Upgrade json-transformer to 1.0.10 Upgrade pojo-generator to 1.7.6
Upgrade metrics to 3.3.2 Upgrade job-manager to 4.3.2 Upgrade framework-jmx-command-client to 2.4.3
Upgrade cpp.platform.system- to 6.4.2 Upgrade cpp.platform.library.system.id-mapper to 6.4.4 Upgrade
cpp.platform.library.audit to 6.4.4 Upgrade cpp.context.access-control to 6.4.9 Upgrade
cpp.platform.library.utils to 6.4.10 Upgrade cpp.platform.library.system.authorisation to 6.4.2
Upgrade unified-search-library to 2.0.2

## [9.4.27] - 2020-04-14

### Changed

- Upgrade framework-generators version to 2.4.2
- Upgrade cpp.platform.library.audit to 6.4.3
- Upgrade cpp.context.access-control to 6.4.8

## [9.4.26] - 2020-04-14

### Changed

- Upgrade event-store version to 2.4.11
- Upgrade cpp.context.access-control to 6.4.6

## [9.4.25] - 2020-04-09

### Changed

- Upgrade file-service version to 1.17.13
- Upgrade framework version to 6.4.1
- Upgrade metrics version to 3.3.1
- Upgrade event-store version to 2.4.10
- Upgrade framework-generators version to 2.4.1
- Upgrade framework-jmx-command-client version to 2.4.2
- Upgrade job-manager version to 4.3.1

- Upgrade cpp.platform.maven.library-parent-pom to 6.4.1
- Upgrade cpp.platform.system-users to 6.4.1
- Upgrade cpp.platform.library.system.id-mapper to 6.4.3
- Upgrade cpp.platform.library.audit to 6.4.2
- Upgrade cpp.context.access-control to 6.4.5
- Upgrade cpp.platform.library.utils to 6.4.9
- Upgrade cpp.platform.library.system.authorisation to 6.4.1
- Upgrade cpp.platform.library.unifiedsearch to 2.0.1

## [9.4.24] - 2020-04-02

### Changed

- Upgrade unified search library to 2.0.0 - Ingest library to stop forcing refresh

## [9.4.23] - 2020-03-04

### Changed

- Upgrade event-store version to 2.4.9

## [9.4.21] - 2020-02-03

### Changed

- Upgrade unified.search-library version to 1.1.57

## [9.4.20] - 2020-01-30

### Changed

- Upgrade event-store version to 2.4.8

## [9.4.19] - 2020-01-27

### Changed

- Upgrade event-store version to 2.4.7

## [9.4.18] - 2020-01-22

### Changed

- Upgrade unified.search-library version to 1.1.56

## [9.4.17] - 2020-01-22

### Changed

- Upgrade event-store version to 2.4.6

## [9.4.16] - 2020-01-17

### Changed

- Upgrade unified.search-library version to 1.1.55

## [9.4.15] - 2020-01-09

### Changed

- Upgrade unified.search-library version to 1.1.54

## [9.4.14] - 2020-01-06

### Changed

- Upgrade event-store version to 2.4.5

## [9.4.13] - 2020-01-05

### Changed

- Upgrade unified.search-library version to 1.1.53

## [9.4.12] - 2020-01-03

### Changed

- Upgrade unified.search-library version to 1.1.52

## [9.4.11] - 2020-01-03

### Changed

- Upgrade unified.search-library version to 1.1.51

## [9.4.10] - 2019-12-19

### Changed

- Upgrade unified.search-library version to 1.1.50

## [9.4.9] - 2019-12-19

### Added

- stream-transformation-tool version to pom
- framework-jmx-command-client version to pom
- stream-transformation-tool-anonymise artifact to dependency management

## [9.4.8] - 2019-12-15

### Changed

- Upgrade unified.search-library version to 1.1.49

## [9.4.7] - 2019-12-14

### Changed

- Upgrade unified.search-library version to 1.1.47

## [9.4.6] - 2019-12-14

### Changed

- Upgrade unified.search-library version to 1.1.46

## [9.4.5] - 2019-12-06

### Changed

- Upgrade event-store version to 2.4.3

## [9.4.4] - 2019-11-25

### Changed

- Upgrade event-store version to 2.4.2

## [9.4.3] - 2019-11-20

### Changed

- Upgrade event-store version to 2.4.1

## [9.4.2] - 2019-11-14

### Changed

- Upgrade unified.search-library version to 1.1.44

## [9.4.0] - 2019-11-13

### Changed

- Upgrade access-control version to 6.4.0
- Upgrade audit version to 6.4.0
- Upgrade system-users version to 6.4.0
- Upgrade id-mapper-client version to 6.4.0
- Upgrade system-authorisation version to 6.4.0
- Upgrade framework version to 6.4.0
- Upgrade metrics version to 3.2.3
- Upgrade event-store version to 2.4.0
- Upgrade framework-generators version to 2.4.0
- Upgrade job-manager version to 4.3.0

### Added

- Add test enveloper provider dependency to dependency management

## [9.3.7] - 2019-11-08

### Changed

- Upgrade access-control version to 6.3.7
- Upgrade audit version to 6.3.6
- Upgrade system-users version to 6.3.4
- Upgrade id-mapper-client version to 6.3.5
- Upgrade system-authorisation version to 6.3.5
- Upgrade framework version to 6.3.0
- Upgrade schema-service version to 1.7.6
- Upgrade metrics version to 3.2.3
- Upgrade event-store version to 2.3.1
- Upgrade framework-generators version to 2.3.0
- Upgrade job-manager version to 4.2.3

## [9.3.5] - 2019-10-30

### Changed

- Upgrade event-store version to 2.2.6

## [9.3.4] - 2019-10-29

### Changed

- Upgrade event-store version to 2.2.5

## [9.3.3] - 2019-10-29

### Changed

- Upgrade event-store version to 2.2.4

## [9.3.2] - 2019-10-25

### Changed

- Upgrade access-control version to 6.3.4
- Upgrade audit version to 6.3.4
- Upgrade system-users version to 6.3.2
- Upgrade id-mapper-client version to 6.3.3
- Upgrade system-authorisation version to 6.3.2
- Upgrade framework version to 6.2.2
- Upgrade file-service version to 1.17.12
- Upgrade utilities version to 1.20.3
- Upgrade jsonschema-pojo-generator to version 1.7.4
- Upgrade schema-service version to 1.7.5
- Upgrade metrics version to 3.2.2
- Upgrade event-store version to 2.2.3
- Upgrade framework-generators version to 2.2.2
- Upgrade json-transformer version to 1.0.9
- Upgrade job-manager version to 4.2.2

## [9.3.1] - 2019-10-18

### Changed

- Upgrade access-control version to 6.3.3
- Upgrade audit version to 6.3.3
- Upgrade system-users version to 6.3.1
- Upgrade id-mapper-client version to 6.3.2
- Upgrade system-authorisation version to 6.3.1
- Upgrade framework version to 6.2.1
- Upgrade metrics version to 3.2.1
- Upgrade event-store version to 2.2.1
- Upgrade framework-generators version to 2.2.1
- Upgrade job-manager version to 4.2.1

## [9.3.0] - 2019-10-15

### Changed

- Upgrade access-control version to 6.3.2
- Upgrade audit version to 6.3.1
- Upgrade system-users version to 6.3.0
- Upgrade id-mapper-client version to 6.3.1
- Upgrade system-authorisation version to 6.3.0
- Upgrade framework version to 6.2.0
- Upgrade metrics version to 3.2.0
- Upgrade event-store version to 2.2.0
- Upgrade framework-generators version to 2.2.0
- Upgrade job-manager version to 4.2.0

## [9.2.1] - 2019-10-01

### Changed

- Add debug logging interceptor to the Command API, Query API and Event Listener. Each component has
  its own interceptor so that fine grain control of logging can be applied in Wildfly.
- Upgrade access-control version to 6.2.0
- Upgrade audit version to 6.2.0
- Upgrade system-users version to 6.2.0
- Upgrade id-mapper-client version to 6.2.0
- Upgrade system-authorisation version to 6.2.1
- Upgrade framework version to 6.1.1
- Upgrade metrics version to 3.1.0
- Upgrade event-store version to 2.1.2
- Upgrade framework-generators version to 2.1.0
- Upgrade job-manager version to 4.1.0

## [9.2.0] - 2019-09-26

### Changed

- Event store now has MI changes which runs Catchup in series for each Subscription
- Event store Indexer Catchup and Event Catchup now run the same code.
- Upgrade event-store version to 2.1.0
- Upgrade system-authorisation version to 6.1.15

## [9.1.19] - 2019-10-08

### Changed

- Upgrade event-store version to 2.0.25

## [9.1.18] - 2019-10-07

### Changed

- Upgrade event-store version to 2.0.24

## [9.1.17] - 2019-09-24

### Changed

- Upgrade system-authorisation version to 6.1.14
- Upgrade event-store version to 2.0.22

## [9.1.16] - 2019-09-23

### Changed

- Upgrade access-control version to 6.1.6
- Upgrade audit version to 6.1.5
- Upgrade system-users version to 6.1.5
- Upgrade id-mapper-client version to 6.1.5
- Upgrade system-authorisation version to 6.1.13
- Upgrade framework version to 6.0.16
- Upgrade metrics version to 3.0.8
- Upgrade event-store version to 2.0.21
- Upgrade framework-generators version to 2.0.12
- Upgrade job-manager version to 4.0.8

## [9.1.15] - 2019-09-23

### Changed

- Upgrade event-store version to 2.0.20
- Upgrade system-authorisation version to 6.1.12

## [9.1.14] - 2019-09-20

### Changed

- Upgrade access-control version to 6.1.5
- Upgrade audit version to 6.1.4
- Upgrade system-users version to 6.1.4
- Upgrade id-mapper-client version to 6.1.4
- Upgrade system-authorisation version to 6.1.11
- Upgrade framework-api version to 4.1.0
- Upgrade framework version to 6.0.15
- Upgrade jsonschema-pojo-generator version to 1.7.3
- Upgrade metrics version to 3.0.7
- Upgrade event-store version to 2.0.19
- Upgrade framework-generators version to 2.0.11
- Upgrade json-transformer version to 1.0.8
- Upgrade job-manager version to 4.0.7
- Upgrade unified.search-library version to 1.1.40

## [9.1.11] - 2019-09-18

### Changed

- Upgrade event-store version to 2.0.18
- Upgrade system-authorisation version to 6.1.9

## [9.1.10] - 2019-09-16

### Changed

- Upgrade event-store version to 2.0.17
- Upgrade system-authorisation version to 6.1.7

## [9.1.9] - 2019-09-15

### Changed

- Upgrade unified.search-library to 1.1.38

## [9.1.8] - 2019-09-13

### Changed

- Upgrade event-store version to 2.0.16
- Upgrade system-authorisation version to 6.1.6

## [9.1.7] - 2019-09-12

### Changed

- Upgrade framework version to 6.0.14
- Upgrade event-store version to 2.0.15
- Upgrade framework-generators version to 2.0.10
- Upgrade access-control version to 6.1.3
- Upgrade audit version to 6.1.3
- Upgrade system-users version to 6.1.3
- Upgrade id-mapper-client version to 6.1.3
- Upgrade metrics version to 3.0.6
- Upgrade system-authorisation version to 6.1.5

## [9.1.5] - 2019-09-08

### Changed

- Upgrade framework version to 6.0.12
- Upgrade event-store version to 2.0.14
- Upgrade framework-generators version to 2.0.9
- Upgrade access-control version to 6.1.2
- Upgrade audit version to 6.1.2
- Upgrade system-users version to 6.1.2
- Upgrade id-mapper-client version to 6.1.2
- Upgrade metrics version to 3.0.5
- Upgrade system-authorisation version to 6.1.4

## [9.1.4] - 2019-09-06

### Changed

- Upgrade event-store version to 2.0.13

## [9.1.3] - 2019-09-04

### Changed

- Upgrade event-store version to 2.0.12

## [9.1.2] - 2019-09-02

### Changed

- Upgrade unified.search-library to 1.1.35

## [9.1.1] - 2019-09-02

### Changed

- Upgrade framework version to 6.0.11
- Upgrade event-store version to 2.0.11
- Upgrade framework-generators version to 2.0.8
- Upgrade access-control version to 6.1.1
- Upgrade audit version to 6.1.1
- Upgrade system-users version to 6.1.1
- Upgrade id-mapper-client version to 6.1.1
- Upgrade metrics version to 3.0.4
- Upgrade unified.search-library to 1.1.33

## [9.0.32] - 2019-07-17

### Added

- Add plugin support for single event-source.yaml for service
- Add plugin support for subscription-descriptor.yaml
- Add plugin support for public-publications-descriptor.yaml

### Changed

- Upgrade common-bom version to 2.2.0
- Upgrade framework-api version to 4.0.1
- Upgrade framework version to 6.0.10
- Upgrade event-store version to 2.0.10
- Upgrade framework-generators version to 2.0.7
- Upgrade access-control version to 6.1.0
- Upgrade audit version to 6.1.0
- Upgrade system-users version to 6.1.0
- Upgrade id-mapper-client version to 6.1.0
- Upgrade metrics version to 3.0.3
- Upgrade raml-maven-plugin version to 1.6.9
- Upgrade file-service version to 1.17.11
- Upgrade utilities version to 1.20.2
- Upgrade test-utils version to 1.24.3
- Upgrade generator-maven-plugin version to 2.7.2
- Upgrade jsonschema-pojo-generator version to 1.7.1
- Upgrade json-schema-catalog version to 1.7.4
- Upgrade metrics version to 3.0.3
- Update Jolt version and change SNAPSHOT to FRAMEWORK-SNAPSHOT

## [9.0.0-RC9] - 2019-07-09

- Change priority for UnifiedSearchIngestionRetryInterceptor

## [9.0.0-RC8] - 2019-07-09

- Update parent-pom to version 2.5.7 to fix enforcer issue

## [9.0.0-RC7] - 2019-07-08

### Changed

- Add UnifiedSearchIngestionRetryInterceptor to event-indexer component

## [9.0.0-RC6] - 2019-07-05

### Changed

- Update event-listener component dependencies

## [9.0.0-RC5] - 2019-07-05

### Changed

- Upgrade unifiedsearch library dependency version to 1.1.11

## [9.0.0-RC4] - 2019-07-04

### Changed

- Upgrade framework version to 6.0.0-RC2
- Upgrade event-store version to 2.0.0-RC3
- Upgrade framework-generators version to 2.0.0-RC2
- Upgrade access-control version to 6.0.0-RC2
- Upgrade audit version to 5.0.0-RC2
- Upgrade system-users version to 5.0.0-RC2
- Upgrade id-mapper-client version to 4.0.0-RC2
- Upgrade metrics version to 3.0.0-RC2

## [9.0.0-RC3] - 2019-07-03

### Changed

- Moved dependencies to event-indexer component

## [9.0.0-RC2] - 2019-07-02

### Changed

- Add event-indexer to bom

## [9.0.0-RC1] - 2019-07-01

### Changed

- Upgrade framework-api version to 4.0.0
- Upgrade framework version to 6.0.0-RC1
- Upgrade event-store version to 2.0.0-RC2
- Upgrade framework-generators version to 2.0.0-RC1
- Upgrade jsonschema-pojo-generator version to 1.7.0
- Upgrade json-schema-catalog version to 1.7.2
- Upgrade common-bom version to 2.2.0
- Upgrade access-control version to 6.0.0-RC1
- Upgrade audit version to 5.0.0-RC1
- Upgrade system-users version to 5.0.0-RC1
- Upgrade id-mapper-client version to 4.0.0-RC1
- Upgrade raml-maven-plugin version to 1.6.8
- Upgrade file-service version to 1.17.9
- Upgrade utilities version to 1.20.0
- Upgrade test-utils version to 1.24.2
- Upgrade test-utils version to 1.24.2
- Upgrade generator-plugin version to 2.7.1
- Upgrade metrics version to 3.0.0-RC1

## [7.1.0] - 2019-01-23

### Changed

- Upgrade framework-api version to 3.1.0
- Upgrade framework version to 5.1.0
- Upgrade jsonschema-pojo-generator version to 1.5.5
- Upgrade metrics version to 2.1.0
- Upgrade framework-domain version to 1.1.0
- Upgrade event-store version to 1.1.2
- Upgrade framework-generators version to 1.1.2
- Upgrade access-control version to 5.1.0
- Upgrade audit version to 4.1.0
- Upgrade system-users version to 4.1.0
- Upgrade id-mapper-client version to 3.1.0
- Upgrade file-service version to 1.17.2
- Upgrade utilities version to 1.16.2

## [7.0.6] - 2019-01-21

### Changed

- Upgrade access-control version to 5.0.6
- Upgrade audit version to 4.0.6
- Upgrade framework-generators version to 1.0.7

## [7.0.5] - 2019-01-18

### Added

- Added maven profile for subscription generation from yaml

### Changed

- Upgrade access-control version to 5.0.5
- Upgrade audit version to 4.0.5
- Upgrade event-store version to 1.0.8
- Upgrade framework-generators version to 1.0.6

## [7.0.4] - 2018-12-21

### Changed

- Upgrade access-control version to 5.0.4
- Upgrade audit version to 4.0.4
- Upgrade system-users version to 4.0.3
- Upgrade id-mapper-client version to 3.0.2
- Upgrade framework version to 5.0.6
- Upgrade metrics version to 2.0.3
- Upgrade framework-domain version to 1.0.5
- Upgrade event-store version to 1.0.7
- Upgrade framework-generators version to 1.0.5

## [7.0.3] - 2018-12-13

### Changed

- Upgrade access-control version to 5.0.3
- Upgrade audit version to 4.0.3
- Upgrade event-store version to 1.0.5
- Upgrade framework-generators version to 1.0.3

## [7.0.2] - 2018-11-20

### Changed

- Upgrade access-control version to 5.0.2
- Upgrade audit version to 4.0.2
- Upgrade system-users version to 4.0.1
- Upgrade id-mapper-client version to 3.0.1
- Upgrade framework-api version to 3.0.1
- Upgrade framework version to 5.0.4
- Upgrade jsonschema-pojo-generator version to 1.5.4
- Upgrade metrics version to 2.0.1
- Upgrade framework-domain version to 1.0.3
- Upgrade event-store version to 1.0.4
- Upgrade framework-generators version to 1.0.2

## [7.0.1] - 2018-11-13

### Changed

- Add event buffer dependency to event-processor component for future support
- Upgrade access-control version to 5.0.1
- Upgrade audit version to 4.0.1
- Upgrade event-store version to 1.0.3
- Upgrade framework-generators version to 1.0.1

## [7.0.0] - 2018-10-09

### Changed

- Upgrade microservices-framework version to 5.0.3
- Upgrade access-control version to 5.0.0
- Upgrade audit version to 4.0.0
- Upgrade system-users version to 4.0.0
- Upgrade id-mapper-client version to 3.0.0
- Upgrade framework.api version to 3.0.0
- Upgrade file-service version to 1.17.1
- Upgrade utilities version to 1.16.1
- Upgrade test-utils version to 1.18.1
- Upgrade jsonschema-pojo-generator version to 1.5.3
- Upgrade schema-service version to 1.4.2
- Upgrade metrics-servlet version to 2.0.0
- Upgrade framework-domain version to 1.0.2
- Upgrade event-store version to 1.0.2
- Upgrade framework-generators version to 1.0.0

## [6.4.3] - 2018-09-25

### Changed

- Upgrade microservices-framework version to 4.4.2 for minor bug fix

## [6.4.2] - 2018-09-20

### Changed

- Upgrade microservices-framework version to 4.4.1 for minor bug fixes

## [6.4.0] - 2018-08-13

### Changed

- Upgrade microservices-framework version to 4.4.0 to add additional properties multipart
- Upgrade utilities dependency version to 1.15.1 to fix additional properties serialization
  nullpointer exception
- Upgrade schema catalog dependency version to 1.4.1 to fix duplicate key exception
- Upgrade pojo generator dependency version to 1.5.2 to fix optionals plugin ordering issue

## [6.3.2] - 2018-08-02

### Changed

- new activiti-platform-library dependency which excludes the JGRAPHX dependency, don't look like we
  are using it so should be safe to exclude. The proper fix is to upgrade activiti, which might be
  more difficult task.

## [6.3.1] - 2018-07-27

### Added

- system documentgenerator client library with version 1.0.4

## [6.3.0] - 2018-07-27

### Changed

- Upgrade microservices-framework version to 4.3.3
- Upgrade utilities dependency version to 1.15.0 to fix integer enum serialization/deserialization
- Upgrade schema catalog dependency version to 1.4.0 to fix resource loading issue

## [6.2.1] - 2018-07-24

### Changed

- Update system users library version to 3.1.1

## [6.2.0] - 2018-07-12

### Changed

- Update dependencies for framework version 4.3.2

## [6.1.1] - 2018-07-06

### Changed

- Update framework to version 4.2.2

## [6.1.0] - 2018-06-22

### Changed

- Updated common-bom version to 2.1.2 to fix apache tika security issues

## [6.0.13] - 2018-06-18

### Changed

- Update service-parent-pom-parent domain-dsl-version to 0.3.1

## [6.0.12] - 2018-06-11

### Changed

- Update access control, system-users and audit library versions

## [6.0.11] - 2018-06-05

### Changed

- Update access control, system-users and audit library versions

## [6.0.10] - 2018-05-22

### Changed

- Update access control, system-users and audit library versions

## [6.0.9] - 2018-05-22

### Changed

- Upgrade microservices-framework version to 4.0.7
- Upgrade raml maven plugin version to 1.6.2

## [6.0.7] - 2018-05-18

### Changed

- Upgrade dependency versions for Jackson 2.8.11 to fix Jackson security issues

## [6.0.2] - 2018-04-12

### Changed

- Updated domain test dsl version to 0.3.0

## [6.0.1] - 2018-04-05

### Added

- Skip event pojo generation property

### Changed

- Updated schema catalog to version 1.2.1

## [6.0.0] - 2018-03-12

### Changed

- Updated framework-api to version 2.0.1
- Updated framework to version 4.0.0
- Updated access control to version 4.0.0
- Updated audit to version 3.0.0
- Updated system user to version 3.0.0

## [5.2.1] - 2018-02-16

### Changed

- Update pojo generator to version 1.3.3
- Added additional properties plugin to pojo generator

## [5.2.0] - 2018-02-14

### Changed

- Update pojo generator and schema catalog to latest versions

## [5.1.0] 2018-01-29

### Changed

- Updated system users and access control libraries

## [5.0.5] 2018-01-24

### Changed

- Update Json Pojo Generator to 1.2.1
- Update System Users to 2.2.4

## [5.0.4] 2018-01-23

### Added

- Dependency for domain DSL tool

### Changed

- Upgraded to framework 3.1.0

## [5.0.3] 2018-01-22

### Changed

- Upgraded to system-users library to 2.2.3

## [5.0.2] 2018-01-22

### Fixed

- Upgraded to parent-pom 2.4.1 to fix missing liquibase property

## [5.0.0] 2018-01-19

### Changed

- Upgraded to parent 2.4.0 for maven site improvements
- Upgraded to use framework-api 1.1.0
- Upgraded to use framework 3.0.0
- Upgraded all libraries to framework 3.0.0 compatible versions
- Added component specific interceptors (moved from framework)
- Added component specific dependencies (moved from framework)

## [4.5.4] 2018-01-12

### Fixed

- Correct schema pojo generation skip property

## [4.5.3] 2018-01-12

### Changed

- Updated framework version to 2.5.3, backwards compatibility fix
- Updated plugins to support both schema pojo and schema catalog generation

## [4.5.0] 2018-01-03

### Changed

- Update Framework to 2.5.1 giving support for Json Schema Catalog
- Update Library Parent Pom to version 3.3.0
- Update Audit to 2.2.0

### Added

- Schema Catalog Generation Plugins.

## [4.4.0] - 2017-12-11

### Added

- POJO generation depencies and plugin

## [4.3.1] - 2017-12-11

### Changed

- Audit version 2.1.0

## [4.3.0] - 2017-12-11

### Added

- File Service 1.14.0
- Utilities 1.11.0
- Test Utils 1.15.0
- Wiremock Test Utils 1.1.0

### Changed

- Upgraded to framework
  version [2.4.3](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-2.4.3)

## [4.2.2] - 2017-11-22

### Changed

- Added VFS URL resolver to controller service components to avoid future confusion

## [4.2.1] - 2017-11-17

### Changed

- Upgraded common BOM to 2.0.30

## [4.2.0] - 2017-11-07

### Changed

- Upgraded to framework
  version [2.4.1](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-2.4.1)
- Upgraded common BOM to 2.0.29
- Upgraded access control to 3.2.0

## [4.1.4] - 2017-11-07

### Changed

- Upgraded system user library to include SJP context

## [4.1.3] - 2017-10-19

### Changed

- Upgraded to framework
  version [2.3.2](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-2.3.2)
- Upgraded common-bom to 2.0.27 for docmosis.version fix

## [4.1.2] - 2017-10-16

### Added

- Framework wildfly-vfs to command-handler and query-view

## [4.1.1] - 2017-10-13

### Added

- Framework wildfly-vfs dependency

### Changed

- Upgraded to framework
  version [2.3.1](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-2.3.1)

## [4.0.22] - 2017-10-06

### Changed

- Upgraded to version 1.2.1 of the activiti library
- Upgraded to framework
  version [2.3.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-2.3.0)

## [4.0.21] - 2017-09-21

No changes

## [4.0.20] - 2017-09-21

### Changed

- Upgraded to version 2.0.6 o fthe system users library

## [4.0.19] - 2017-09-19

### Added

- Added activiti-embedded-rest

## [4.0.18] - 2017-09-19

### Changed

- Upgraded to version 2.0.5 of the system users library

## [4.0.17] - 2017-09-15

### Changed

- Upgraded to version 2.2.1 of the authorisation library, to fix a problem with conflicting
  capability manifests in single war deployments

## [4.0.16] - 2017-09-04

### Changed

- Upgraded to version 2.0.4 of the system users library

## [4.0.15] - 2017-09-01

### Changed

- Reverted to Framework 2.2.1 to exclude potential breaking changes

## [4.0.12] - 2017-08-18

- Upgrade Authorisation BOM to 2.2.0

## [4.0.10] - 2017-08-17

- Upgraded common-bom to 2.0.23 for commons-collections and postgres

## [4.0.9] - 2017-08-14

- Upgraded common-bom to 2.0.22 for test-utils dependency

## [4.0.8] - 2017-08-03

- Upgraded common-bom to 2.0.21 for file-service liquibase OID fix

## [4.0.7] - 2017-08-02

- Upgraded to framework
  version [2.2.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-2.2.0)

## [4.0.6] - 2017-07-27

- Upgrade access control version to 3.1.1

## [4.0.5] - 2017-07-21

- Upgrade Authorisation BOM to 2.1.1

## [4.0.4] - 2017-07-19

- Upgrade Authorisation BOM to 2.1.0

## [4.0.3] - 2017-07-18

- Add Authorisation BOM

## [4.0.1] - 2017-07-04

### Changed

- Upgrade Common BOM to 2.0.20 to bring in file service connection leak fix

## [4.0.0] - 2017-06-27

### Changed

- Upgraded to framework
  version [2.0.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-2.0.0)
- Upgraded all libraries to versions rebuilt with framework 2.0.0
- Added ID mapper client library
- Renamed service component POMs
- Added exclusion to RAML generator configuration to prevent messaging clients being generated for
  emitted events of the component being sent to. Clients should only be generated for messages that
  the destination consumes, not what it emits.

## [3.0.29] - 2017-06-19

### Changed

- Upgrade Common BOM to 2.0.19 to upgrade Activiti

## [3.0.28] - 2017-06-08

### Changed

- Revert to Wiremock utils 1.1.0 to avoid breaking change
- Upgrade access control, audit client and system users to latest versions

## [3.0.27] - 2017-06-07

### Changed

- Upgraded to framework
  version [1.7.1](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.7.1)
  .

## [3.0.25] - 2017-05-16

### Changed

- Upgraded to framework
  version [1.7.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.7.0)
  .
- Upgrade system users library to 1.0.15

## [3.0.22] - 2017-05-05

### Changed

- Upgraded to framework
  version [1.6.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.6.0)
  .

## [3.0.21] - 2017-05-03

### Changed

- Upgraded to framework
  version [1.5.2](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.5.2)
  .

### Fixed

- Disable "add library to classpath" by *default* in maven-war-plugin

## [3.0.20] - 2017-04-13

### Changed

- Upgraded to common-bom 2.0.16 for addition of commons-compress library dependency

## [3.0.19] - 2017-04-10

### Changed

- Upgraded to the new version of the system-users library that contains the IDPC and the
  authorisation service system user ids

## [3.0.18] - 2017-03-30

### Changed

- Upgraded to framework
  version [1.4.3](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.4.3)
  .

## [3.0.17] - 2017-03-09

### Changed

- Upgraded the parent POM to 2.0.15 for webdav jackrabbit

## [3.0.16] - 2017-03-09

### Changed

- Upgraded the parent POM to 2.0.14

## [3.0.15] - 2017-03-09

### Changed

- Upgraded to framework
  version [1.4.2](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.4.2)
  .

## [3.0.14] - 2017-03-08

### Changed

- Upgraded to raml-maven-plugin 1.4.0
- Upgraded to common-bom 2.0.16
- Upgrade access control library version to 2.0.32

## [3.0.13] - 2017-02-21

### Changed

- Upgraded to common-bom 2.0.14 to include fixed File-Service 1.1.0 version dependency

## [3.0.12] - 2017-12-??

### Added

- Added activation mechanism for the JsonSchemaValidatingMockMaker

### Changed

- Upgraded to framework
  version [1.4.1](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.4.1)
  .
- Upgraded to common-bom 2.0.13 to include File-Service

## [3.0.11] - 2017-02-03

### Changed

- Upgraded to framework
  version [1.4.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.4.0)
  .
- Upgraded to access control version 2.0.31 to fix causation issue in initial set of providers.

## [3.0.10] - 2017-01-26

### Changed

- Upgraded to framework
  version [1.3.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.3.0)
  .

## [3.0.9] - 2017-01-24

### Changed

- Upgrade access control library version to 2.0.30

## [3.0.8] - 2017-01-23

### Changed

- Upgraded to framework
  version [1.2.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.2.0)
  .

## [3.0.7] - 2017-01-09

### Added

- Add release profile configuration to include EJB delivery active setting from common resources JAR

## [3.0.6] - 2017-01-09

### Changed

- Upgrade system users library to 1.0.10

## [3.0.5] - 2017-01-09

### Changed

- Upgraded the cpp.audit.version property version to [0.1.11]

## [3.0.4] - 2016-12-23

### Changed

- Upgraded to framework
  version [1.0.1](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.0.1)
  . This fixes a problem with metadata parsing in the logging filter.

## [3.0.0] - 2016-12-16

### Changed

- Upgraded to framework
  version [1.0.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-1.0.0)
  . This requires database schemas to be upgraded and is therefore not backwards compatible. The
  upgrade switches on aggregate snapshots by default.

## [2.0.49] - 2016-12-07

### Changed

- Upgraded the parent-pom version to [2.0.13] to use a fixed version of the RAML dependency enforcer
  rule

## [2.0.48] - 2016-12-06

### Changed

- Upgraded the parent-pom version to [2.0.12] to enforce RAML dependencies to use the latest version

## [2.0.47] - 2016-12-06

### Changed

- Make RAML messaging client generator to process public-event.messaging.raml's prefixed with
  context name

## [2.0.46] - 2016-11-30

### Changed

- Updated parent POM to 2.0.11

## [2.0.45] - 2016-11-24

### Changed

- Upgraded the framework version
  to [0.35.0] (https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.35.0)

## [2.0.43] - 2016-11-08

### Changed

- Upgraded the common-bom version to [2.0.12] (Removed json-restassured dependency)

## [2.0.42] - 2016-09-31

### Changed

- Upgraded framework version
  to [0.33.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.33.0)
  with new handler matchers and REST poller.

## [2.0.41] - 2016-09-21

### Changed

- Upgraded framework version
  to [0.32.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.32.0)
  . This introduces aggregate snapshots as an optional feature, adds matchers for testing JSON
  envelopes, and fixes an issue with merging metadata from headers and a REST Payload.

## [2.0.40] - 2016-10-20

### Changed

- Upgraded common-bom version to 2.0.11

## [2.0.37] - 2016-10-12

### Changed

- Upgraded audit library version to 0.1.8

## [2.0.36] - 2016-10-11

### Added

- New service-components modules for providing encapsulated dependency groupings for each service
  component (i.e. wraps framework with libraries)
- New parent pom aggregator to manage new module structure
- Add new system-users library dependency management at version 1.0.7
- README.MD for documenting the two-phase release process required to manage the new _
  cpp.service.components.version_ property

### Changed

- Upgraded common-bom version to 2.0.8
- Upgraded access-control library version to 2.0.19
- Upgraded audit library version to 0.1.7
- service-parent-pom configuration to remove duplicates

## [2.0.32] - 2016-09-28

### Changed

- Upgraded common-bom version to 2.0.6 (added quartz dependencies, which is used by scheduler
  service)

## [2.0.31] - 2016-09-23

### Changed

- Upgraded common-bom version to 2.0.5 (added slf4j-simpl and httpmime dependencies, which are used
  by integration tests)

## [2.0.30] - 2016-09-21

### Changed

- Upgraded framework version
  to [0.28.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.28.0)
- Upgraded audit library version to 0.1.6
- Upgraded access-control library version to 2.0.17

## [2.0.29] - 2016-09-21

### Added

- Audit bom dependency import
- Exclusion on drools package for jacoco plugin

### Changed

- Upgraded framework version
  to [0.27.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.27.0)
- Upgraded audit library version to 0.1.5
- Upgraded access-control library version to 2.0.16

## [2.0.28] - 2016-09-16

### Changed

- Upgraded common-bom version to 2.0.4 (removing active MQ dependencies)

## [2.0.26] - 2016-09-09

### Changed

- Upgraded framework version to 0.24.0
  - https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.24.0

## [2.0.17] - 2016-08-11

- Upgraded framework
  to [0.15.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.15.0)
  Add a simple Audit Client Add a toDebugStringPrettyPrinted() method that returns the JsonEnvelope
  as JSON

## [2.0.16] - 2016-08-10

## Changed

- Upgraded BOM version to 2.0.1 (introduces Apache Commons CSV)

## [2.0.15] - 2016-08-08

## Changed

- Update to Wiremock service 1.1.0 and add dependency management

## [2.0.14] - 2016-07-29

## Changed

- Upgraded to
  framework [0.13.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.13.0)
  . This adds a service component passthrough test utility, support for listening to all events on a
  topic or queue, removes the limitation on messaging RAML resources to allow any queue or topic
  name and adds the causation UUID list to an HTTP header when sent by the REST client.

## Added

- Import access control BOM 2.0.5 for dependency management

## [2.0.12] - 2016-07-19

### Changed

- Updated parent POM to 2.0.6
- Updated common resources to 1.0.2

## [2.0.11] - 2016-07-19

### Changed

- Upgraded to
  framework [0.12.3](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.12.3)
  . This fixes events store retrieval ordering and 403 handling for REST clients.
- Updated parent POM to 2.0.5

### Added

- Added maven war plugin & remote resources plugin configuration (moved from parent pom)

## [2.0.10] - 2016-07-18

### Fixed

- Corrected messaging client generation to specific RAML file patterns to avoid clients being
  generated from local RAML files unless they match specific patterns for public and private events

## [2.0.9] - 2016-07-15

### Changed

- Updated parent POM to 2.0.4 from new repository
- Update common BOM to 2.0.0 from new repository, including removal of redundant dependencies
  including Spring and Camel

## [2.0.8] - 2016-07-14

### Changed

- Upgraded to
  framework [0.12.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.12.0)

## [2.0.7] - 2016-07-12

### Changed

- RAML plugin configuration changed to generate messaging clients for emitted events described in
  RAML in src/raml. This adds a requirement for the _cpp.service-component_ property to be set in
  any module that contains messaging RAML.

## [2.0.6] - 2016-07-11

### Added

- _wiremock.service.version_ property for Wiremock service dependencies

## [2.0.5] - 2016-07-11

### Fixed

- Correct base package name for messaging client generator

## [2.0.4] - 2016-07-11

### Changed

- Upgrade to framework
  version [0.11.0](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.11.0)

## [2.0.3] - 2016-07-05

### Changed

- Upgrade to use
  framework [0.10.1](https://github.com/CJSCommonPlatform/microservice_framework/releases/tag/release-0.10.1)
- Convert to use framework POM

## [2.0.2] - 2016-06-24

### Added

- Initial release of settings from original service-parent-pom
