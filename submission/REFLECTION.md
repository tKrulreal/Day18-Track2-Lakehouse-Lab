# Reflection: Lakehouse Anti-Patterns

Among the "Top 5 Lakehouse Anti-Patterns", our team is most vulnerable to **"The Streaming Small-File Accumulation & Uncommitted Orphan Proliferation"**.

### Rationale:
1. **Small-file explosion:** Our real-time ingestion pipelines commit micro-batches every few seconds. Without automated compaction jobs (`OPTIMIZE / rewrite_data_files`), file counts quickly surge into the millions. This creates severe metadata bloat and exponentially inflates object storage request costs (S3 GETs dominate the monthly bill rather than data volume).
2. **Invisible orphan files:** As demonstrated in NB6, uncommitted files left behind by failed or preempted spot-instance writers are never tombstoned in the transaction log. Consequently, standard `VACUUM` commands completely bypass them regardless of retention settings. Without a dedicated sweep mechanism computing the set difference between storage objects and live catalog metadata, these zombie files accumulate indefinitely, incurring silent storage costs and compliance risks.
