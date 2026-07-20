## Copilot instructions for Amazon FSx for NetApp ONTAP documentation

### Repository overview
Product: Amazon FSx for NetApp ONTAP in NetApp Workload Factory

Amazon FSx for NetApp ONTAP is a fully managed AWS service that provides cloud-based file storage with advanced ONTAP data management capabilities. This repository documents FSx for ONTAP as the *Storage* workload component within NetApp Workload Factory.

### Repository structure
- `_include/` – Shared AsciiDoc include snippets reused across topics (file system creation steps, replication initialization, cutover procedures)
- `_whatsnew/` – Release notes content files, one per release date, included by `whats-new.adoc`
- `media/` – Images and screenshots referenced in content files
- `store-redirects/` – Redirect stub files for pages that have been renamed or moved

All primary content (`.adoc` files) lives at the repository root. There is no subdirectory hierarchy for topics.

### Product-specific context

**Architecture and components:**
- *FSx for ONTAP file system*: The top-level AWS resource; contains one or more storage VMs and supports Single AZ or Multiple AZ (HA) deployments
- *Storage VM (SVM)*: An isolated virtual file server within a file system; also called *vserver* in ONTAP; has its own administrative credentials (`vsadmin`) and endpoints; clients mount volumes, CIFS/SMB shares, or iSCSI LUNs via the SVM endpoint
- *Volume*: A virtual storage container within an SVM; volume data primarily consumes SSD storage; types are *FlexVol* (single-node) and *FlexGroup* (scale-out NAS with multiple constituent volumes)
- *Link*: An AWS Lambda function deployed in the customer's AWS account that creates a trust relationship between Workload Factory and an FSx for ONTAP file system; links send ONTAP REST API calls directly to the file system, enabling operations beyond what the native Amazon FSx API supports; one link can be associated with multiple file systems but each file system can only be associated with one link per NetApp account
- *Codebox*: Workload Factory's built-in automation tool that generates REST API, CloudFormation, and Terraform code for operations performed in the console

**Key concepts:**
- *Snapshot*: A read-only, point-in-time copy of a volume; snapshots are the foundation of all backup and replication methods
- *Replication*: SnapMirror-based data replication that creates a secondary copy of a volume on another FSx for ONTAP file system, an on-premises ONTAP system, or Cloud Volumes ONTAP; replicated (data protection) volumes follow the naming format `{OriginalVolumeName}_copy`
- *ARP/AI*: NetApp Autonomous Ransomware Protection with AI; monitors NAS (NFS/SMB) and SAN (iSCSI) environments for abnormal activity and automatically creates immutable snapshots when a threat is detected
- *Immutable files (SnapLock)*: WORM (write-once-read-many) protection for volumes; two retention modes are *Enterprise* (admin can delete before retention expires) and *Compliance* (no deletion before expiry); enabled only at volume creation, cannot be disabled
- *Cache volumes (FlexCache)*: Read-cache volumes that provide local access to data stored in a remote origin volume, improving read performance
- *S3 access point*: An attachment on NFS or SMB/CIFS volumes that allows file data to be accessed via AWS S3 APIs; enables integration with AWS GenAI, ML, and analytics services
- *Configuration analysis*: Workload Factory's daily automated assessment of file system configurations against the AWS Well-Architected Framework pillars: reliability, security, operational excellence, cost optimization, and performance efficiency
- *igroup*: An initiator group used to manage iSCSI host access to block storage LUNs
- *Block device*: An iSCSI-based storage resource backed by a volume; block access is restricted to scale-out deployments with six HA pairs or fewer

**Naming conventions and terminology:**
- The full product name is *Amazon FSx for NetApp ONTAP*; the accepted shortened form is *FSx for ONTAP* (not "FSx ONTAP" or "FSx for NetApp ONTAP")
- File system administrator account: `fsxadmin`; SVM administrator account: `vsadmin`
- Workload Factory uses *Quick create* and *Advanced create* as deployment modes for file system creation
- Tiering policies are shown in Workload Factory with use-case-based names with FSx for ONTAP names in parentheses (for example, *Balanced (Auto)*)
- A *data protection (DP) volume* is a read-only replicated volume; the naming convention is `{SourceVolumeName}_copy`
- *Cascading replication*: A replication chain where a secondary (DP) volume is itself replicated to a tertiary volume (source → secondary → tertiary); a fourth hop from tertiary is not supported
- Storage tiers: *SSD storage tier* (primary, high performance) and *capacity pool storage tier* (secondary, lower cost for cold data)
- *ARP/AI* is always spelled out as "NetApp Autonomous Ransomware Protection with AI (ARP/AI)" on first use
- *SnapLock* is the underlying ONTAP feature for immutable files; documentation refers to this feature as *immutable files powered by SnapLock*

**Technical constraints:**
- Many advanced features require a *link* to be associated with the file system; these include replication management, SMB/CIFS and NFS provisioning, iSCSI volume management, snapshot policy management, ARP/AI, volume autogrow, clone management, and configuration analysis
- iSCSI block access is only available for scale-out deployments with six HA pairs or fewer
- Immutable files (SnapLock) cannot be disabled after enabling and is only configurable at volume creation
- ARP/AI is not supported on NVMe volumes; iSCSI volume support requires a minimum ONTAP version (see repository docs for current requirement)
- Replication supports FSx-to-FSx (same generation), FSx-to-on-premises ONTAP, and FSx-to-Cloud Volumes ONTAP
- Cutover during migration requires that source and target systems run the same major ONTAP version

### Typical user workflows

**Initial setup:** Log in to Workload Factory → Add AWS credentials and permissions → Create FSx for ONTAP file system (Quick create or Advanced create) → Create storage VM → Create volumes

**Enable data protection:** Associate a link with the file system → Enable ARP/AI for the file system → Create or assign a snapshot policy → Set up cross-region replication → Configure volume backup schedule

**Storage VM migration:** Create replication relationship (select Migration use case, enable Replicate storage VM configuration) → Initialize replication relationship (baseline transfer) → Verify data sync → Stop client access → Cut over replication → Update configuration settings on the target SVM

**Explore storage savings:** Navigate to Storage in Workload Factory → Use the storage savings calculator to compare Amazon EBS, EFS, and FSx for Windows File Server costs against FSx for ONTAP

**Set up block storage:** Create a storage VM → Create a volume with block access → Create an igroup → Create a block device → Map igroup to block device → Connect iSCSI clients
