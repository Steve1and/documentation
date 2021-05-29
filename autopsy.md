# Autopsy Basics
***

- Use a PostgreSQL setup for the case database for a multi-user environment.
  - 2 dedicated servers
  - central NAS storage
  - Install PostgreSQL and ActiveMQ on one server.
  - Install Solr on the other
  - Ensure all clients can access central storage at the same UNC or drive letter path.

## Cases and Data Sources

Each case created makes its own folder and has the following contents:
  - `autopsy.db`: SQLite database that will store basic case info and data source info.
    - Unless it is a multi-user case
  - **Export Folder**: Default location to store exported files.
  - **Reports Folder**: Default location to store reports.
  - **ModuleOutput Folder**: Default location for modules to write ouput to.

Autopsy supports the following data sources:
  - Disk Images
  - Local Drives
  - Local Files (Logical Files/Folders)
  - Output from Autopsy Logical Imager
  - Unallocated Space Files (no structure)

Information stored in the Database:
  - File Metadata
    - Name and parent folder path
    - Times (modified, created, etc.)
    - Size
    - MD5
  - Partition Layouts
  - NOTE: The database doesn't have copies of the file content. Just metadata, so it stays fairly small.

Supported Disk Image Formats:
  - Raw (dd) single and split
  - E01
  - Raw disk images of phones (Android)
  - Virtual machine formats

Process to add a Disk image:
  - Point autopsy at the first image, it does the rest.
  - Example: E01 -> E02, E03, etc.
  - NOTE: does not validate E01 files directly on import.

Disk Image Analysis
  - Uses The Sleuth Kit (TSK) to analyze the contents of the image.
  - Detects Volume Systems that break the disk into partitions
  - Detects File Systems that organize a partition so that files can be stored.

Volume systems organize the disk image into one or more volumes (or partitions).
  - Located near the beginning of the disk image
  - Each volume is analyzed to look for a file system.
  - If no volume is found, the entire image is analyzed for a file system. (i.e. flash media).

Autopsy / TSK supports
  - DOS
  - GPT
  - Mac
  - BSD
  - Solaris

File Systems allow files to be stored.
  - Typically located at the beginning of disk image or inside volume.

Autopsy supported formats:
  - NTFS
  - FAT, ExFAT
  - HFS+
  - ISO9660
  - EXT 2/3/4
  - YAFFS2
  - UFS

Orphan files are those that are deleted and no longer have a parent folder.
  - They are accessible in the "`$OrphanFiles`" folder.
  - Finding orphan files in FAT file systems is time intensive.
    - Every cluster must be read and analyzed.
    - Can be disabled when image is added.

Carving recovers deleted files without relying on file system knowledge
  - Relies on file structure internals.
    - JPEG
    - PDF
    - Word Documents
    - EXE
  - Needed when file system doesn't have pointers to file content anymore.
  - Autopsy uses the PhotoRec tool for carving
    - It runs on unallocated space.
    - Carved files are added into a special "`$CarvedFiles`" folder.
    - NOTE: This is technically note done when the data source is added. It is done with the ingest modules.
  - Unallocated space in Autopsy is represented as files.
    - Directly in a volume or in a `$Unalloc` folder.
    - Name based on: `Unalloc_ParentID_StartByte_EndByte`

Disk Image Limitations
  - No RAID
  - No Logical Volume Management
  - No Bitlocker

Local Drive Data Sources:
  - Local drives are C: or E: drives
  - Can by analyzed:
    - Preview a live system (i.e. triage)
    - USB-attached device (write blocker)

Local Drive Analysis:
  - Same process as disk images:
    - Scan for Volumes
    - Scan for File Systems
    - Add files to database
  - Will need to run as administrator to access all drives
  - Analysis cannot be performed if USB device is removed
  - Autopsy can make a 'sparse VHD' image when analyzing a local drive.
    - Useful for triage situations
    - When data is read, Autopsy determines if it has seen that sector before. If not, it saves a copy.
    - If you analyze the entire drive, you'll get a complete image.
    - Choose "Make a VHD..." option
    - By default it will pick a file name in the case folder.
    - Choose the "Update Database..." option if you want Autopsy to update the case database to refer to the VHD file after it is complete.
      - Otherwise, it will have a reference to `\\.\E:` (or something similar).
***
## UI Basics

Tree Structure has 5 top-level nodes:
  - **Data Sources**: Shows the data in the case organized by drive layout and directories.
    - Allows you to find specific files by path.
  - **Views**: Shows files organized by metadata; same data as in the "Data Sources" area, just organized differently.
    - Organized by Images, Documents, Executables, etc.
    - Can organize by extension or by file signature (MIME type).
      - Extension filtering can be done immediately
      - File type filtering requires all files to be analyzed.
  - **Results**: Shows the results from the analysis ingest modules.
    - Has lower-levels of organization
    - Extracted content contains most
    - Specialized ones are separated out.
  - **Tags**: shows what files and results have been tagged.
  - **Reports**: shows the reports generated by the user or a module.

Display Settings grouping can be changed by clicking on the gear at the top right of the tree tab.

When you select a node in the tree, its contents are shown in the upper right, with a table view shown by default.
  - To search within the table, select any row and start typing.

Many rows have 3 columns to give a quick context about an item.
  - **First Column, Score**:
    - If the red exclamation mark icon is there, it means the item is notable.
      - Hash hit
      - A notable tag was applied
    - If the yellow down arrow icon is there, it means the item is suspicious.
      - A module marked it as interesting
      - A normal tag was applied
  - **Second Column, Comment**:
    - A yellow page is shown if the file has a comment.
    - Either in the current case or a past case (requires central repository)
  - **Third Column, Occurrences**:
    - How often this item has been seen in past cases (requires central repository).

The Thumbnail tab can be used to see images and videos.

The content viewer area is in the lower right-hand corner of the screen and shows the contents that Autopsy can discern from a selected item. Elements include:
  - **Hex Viewer**: Displays file content in a basic hex editor
    - Always enabled if file has content
    - Can launch HxD if you need more powerful features.
  - **Text**: Contains various "sub viewers" that are all related to text.
    - **Strings**: shows data that could be text in a given encoding; can have false positives.
    - **Indexed Text**: Shows what is in the keyword search index.
      - For supported file types, shows text after parsing structure.
        - It knows about PDF vs. DOC
      - For unsupported file types, it shows strings.
    - **Translation**: Uses Google or Bing translate (if configured)
      - Translates the same text as shown in "Indexed Text".
  - **Application**: File type-specific viewer:
    - Pictures
    - Video
    - SQLite
    - HTML
    - Registry
    - Binary PList
  - **Message**: Email-like display to show email and text messages; header information and text, RTF, and HTML messages.
  - **File Metadata**: shows metadata about a selected file
    - Similar to data to what is in the table, but easier to copy and paste.
  - **Results**: shows all analysis results performed on the selected item.
    - Shows all data extracted from the selected item.
    - Data comes from the database blackboard.
  - **Annotations**: Shows all examiner-specified data, such as tags and comments.
    - Shows data from both current and past cases (if central repository is enabled).
  - **Other Occurrences**: Shows where else the selected item was seen.
    - Occurrences on other data sources in the same case, or past cases if central repository is enabled.

The Ingest Inbox (The mail button in the top bar of the program) shows users what has been found in background tasks.

To search a database for files by metadata, go to Tools -> File Search by Attributes.
  - You can now choose various search options to go by.
  - NOT A KEYWORD SEARCH

Files can be extracted by right-clicking on them and selecting `Extract File(s)`. These files are saved in the case folder under the Extractions folder.

Other Interfaces are available from the main UI:
  - **Timeline**: Displays events sorted by time
  - **Image Gallery**: Displays pictures and videos grouped by folder
  - **Communications**: Displays accounts, messages, call logs, etc.

***
## Analyzing Data Sources
Ingest modules are plug-in modules responsible for analyzing the data on the drive.
  - Hashing
  - Keyword Searching
  - Web activity
  - Registry analysis
  - File type identification
  - Extension mismatch

Two types of ingest modules:
  1. **File Ingest Modules**: hash calculation, hash lookup, EXIF extraction, add text to keyword index, etc.
    - Are run on every file in the data source, unless "ingest filters" are applied.
    - Many can run in parallel
  2. **Data Source Ingest Modules**: web browser analysis, registry analysis, etc.
    - Run on an entire data source
    - Better for more focused analysis techniques
    - For example:
      - Query database for a set of files
      - Analyze the set of files
      - Post results to blackboard
    - Versus file-level modules:
      - Review every file to determine if it is relevant
      - Analyze the relevant files
      - Post results to blackboard

File Prioritization
  - The Ingest Manager runs in the background and schedules the files for analysis
  - Files go down the pipeline based on priority:
    - User Folders
    - Program Files and other root folders
    - Windows folder
    - Unallocated Space
  - If a 2nd image is added while the 1st is still being ingested, the two are done in parallel.
    - Files from image 2 could be highest priority.

To run ingest modules, either do it from the initial adding of a data source, or right click on the data source in the tree and select "Run Ingest Modules".

Blackboard artifacts are where ingest modules save their results as Blackboard artifacts.
  - Artifacts have a type and one or more attributes, such as:
    - Web Bookmark
    - Hash Hit
    - Encryption Detected
  - Autopsy comes with many predefined types and modules can make their own.
  - Each artifact has one or more attributes.
  - An attribute is a type and value pair
  - Example of a Web Bookmark artifact:
    - URL: http://www.autopsy.com
    - DATETIME: April 1, 2020
  - Modules can choose what attributes they add to an artifact.
  - To view artifacts look under the "Results" option in the tree under "Extracted content".

***
## Hash Lookup Module
The Hash Lookup Module performs the following functions:
  1. Calculates MD5 hash of files
  2. Stores hash in the case database
  3. Looks hash up in hash set
  4. Marks file accordingly as:
    - Known (NSRL) -> could be good or bad
    - Known Bad / Notable

Reasons to use:
  - To include MD5 hash values in your Reports
  - To make ingest faster and skip known files using NSRL
  - To hide known files from UI
  - To identify notable ('known bad') files
  - To keep central repository up to date and correlate with past cases

Hash Calculation Step
  - Skips the Unallocated space files
  - Calculates MD5 of file content
  - Stores hash in the case database
  - If file already has a hash, it does not calculate it again

Hash Set Lookup Step
  - Looks up MD5 in all of the configured hash sets:
    - Does not stop at first hit
  - Supported hash sets:
    - NIST NSRL
      - Flagged as 'KNOWN'
    - EnCase
    - The Sleuth Kit SQLite format (.kdb files)
    - md5sum
    - Hashkeeper

Known status of a File
  - Every file has a "Known Status":
    - Notable /Known Bad
    - Known
    - Unknown - the default

What can be configured:
  - The database to use
  - The hash sets to use
  - To always calculate hash values

Results can be seen under "Results" in the "Hashset Hits" section.

Configuring Hash Sets
  - Open Hash Lookup Module Settings:
    - Tools -> Options
    - "Global Settings" button
    - Either create a new hash set or import one.

Index for NSRL can be downloaded from sleuthkit.org
***
## Simple Modules

**File Type Identification**: Determines file type based on signatures
  - Identifies JPEG based on file starting with `0xffd8`
  - Stores type in database for other modules to use.
  - No configuration needed.
  - File-level ingest module
  - Uses Tiak open source library
  - Detects hundreds of file types and reports them as a MIME type.
    - application/octect-stream -> means unknown type
    - application/zip -> zip file
    - image/jpeg -> jpeg image file
    - audio/mpeg -> audio file
  - Results can be viewed in the File Metadata tab
  - Custom file types can be defined using: Tools -> Options -> File Types
    - Specify:
      - MIME Type (you can make one up)
      - Offset of signature
      - Signature (in bytes or ASCII string)
      - If you want to be alerted if the file type is found.

**File Extension Mismatch Module**: Compares file's extension and file type.
  - Flags if extension is not associated with type.
  - Configuration:
    - File types to focus on
    - Skip known files (default: true)
    - List of extensions for each file type
  - Can produce a lot of false positives.
    - Lots of files get renamed to `.tmp` or `.bak`.
    - Reduce false positives by focusing on file types: Default is multimedia and executables only.
  - Results under the Extracted Content section in the Results tree under Extension Mismatch Detected.

**Exif Module**: Extracts Exif structure from JPEGs
  - Stores metadata in blackboard
  - Can be used to identify camera type used to take a picture, identify the time a picture was taken, or identify geo-coordinates where a picture was taken.
  - No configuration needed.
  - Results under EXIF Metadata in the Extracted Content section of the Results.

**Embedded File Extractor**: Extracts embedded files
  - Opens ZIP, RAR, and other archive files.
  - Extracts images from Office and PDF files.
  - No configuration needed.
  - Extracted files are saved inside of case folder.
  - Added back into the ingest pipelines for analysis with the same priority as a parent file
  - Flagged if it was password protected.

**Email Module**: searches for MBOX, PST, and EML files
  - adds e-mail artifacts to blackboard
  - adds attachments as children of the messages
  - groups them into threads
  - no configuration required
  - Results can be viewed under Communications or under E-Mail Messages in Extracted Content in Results

**Interesting Files Module**: Flags files and folders that you think are interesting.
  - Can be used to always alert and notify when files are found.
  - Useful for automating searches for things like iPhone backups, VMWare images, BitCoin wallets, Cloud storage clients, etc.
  - Example Rule Set:
    - Set Name: VMWare
    - Rule 1:
      - Type Files
      - Full Name: vmplayer.exe
      - Name: Program EXE
    - Rule 2:
      - Type: Files
      - Extension: vmdk
      - Name: VMDK File

**Encryption Detection Module**: Flags files and volumes that are or could be encrypted
  - Detects: Passwords on Office docs and Access DBs; Possibly encrypted files or volumes (high entropy (random), multiple of 512 bytes, and no known file type).
  - Results in: Encryption Detected or Encryption Suspected in Extracted Content in Results.

**Plaso Module**: Uses the open source Plaso tool to parse various logs and file types to extract out time stamps.
  - Some of its time stamps duplicate what Autopsy already extracts
  - It can take a considerable amount of time to run, so it is disabled by default.
  - The following settings are disabled by default:
    - Registry time stamps
    - PE headers (executables)

**Virtual Machine Extractor Module**: Analyzes virtual machines found in a data source.
  - Detects vmdk and vhdi files in a data source
  - Makes a local copy of them
  - Feeds them back in as data sources
  - No configuration needed
  - Results will be listed under Data Sources

**Data Source Integrity Module**: Validates and calculates hash of disk image
  - Ensures integrity of evidence
  - Retrieves hash from E01 or what user entered when data source was added
  - Calculates current hash
  - Generates an alert if they are different
  - Saves hash if there was not one already
***
## Recent Activity Module

What it does:
  - Extracts "recent user activity"
  - Web Activity:
    - Bookmarks, cookies, downloads, etc.
  - Registry Analysis
    - USB Devices
    - User Accounts
    - Installed Programs
    - Programs Run
    - Uses RegRipper to analyze registry Hives
    - Open source tool: http://code.google.com/p/regripper/
    - Engine has modules for different hives
    - Modules analyze registry contents and display results
    - It is not an interactive registry analysis tool
  - Recycle Bin Analysis

Results can be found in the Extracted Content in the Results Tree

No Configuration Options needed.
  - Module is either on or off.

Registry Analysis artifacts:
  - Connected USB devices:
    - Device make, model, ID, and date.
  - Installed Programs
    - Name and install date
  - Programs that were run
  - User Account information

Process for registry analysis:
  - Query for hives in `system32\config\`
***
## Keyword Search Module

What it does: Updates and searches a text index to enable text-based searching

Similar concept to the hash set index.
  - A text index contains a sorted list of words.
  - Each word has a list of documents that it was found in.
  - When adding documents to an index:
    - Words are added if they do not already exist
    - The document ID is added to each word
  - When searching an index:
    - The specified word is found in the sorted list

Autopsy uses Apache Solar as its search engine
  - Popular open source search engine
  - Each case gets its own index
  - The index is stored inside the case folder
  - Will contain:
    - File Names
    - Text from file content
    - Text from artifacts

Creating an Index:
  - The ingest module is responsible for adding ext to the index
  - Ignores 'known' (NSRL) files
  - Intelligently extracts text from each file
    - It knows a PDF vs. DOCX
  - Solr breaks the text into words and updates the index

Autopsy relies on Apache Tika for many common file formats
  - Supports:
    - Office, PDF, OpenDoc, RTF, etc.
    - Metadata from Audio, Video, etc.

HTML Extraction:
  - Most HTML text extractors focus on the content.
  - We want to search comments and java script

Strings Extraction:
  - If the file type is not known to Tika (or it is corrupt) a generic strings extraction approach is used:
    - Look for bytes that could be a string in some language.
  - Two settings:
    - Encodings
    - Languages
  - The more encodings and languages added, the more false positives generated.
  - From Tools -> Options or "Global" button during ingest. Click on the String Extraction Tab

Text Normalization
  - Search engines sometimes normalize Text
  - Autopsy will:
    - Make all searches case insensitive
    - Normalize Unicode sequences.
      - Half-width vs. Full-width characters (typical in Asian languages)
      - Accents stored as a single value vs. two

Searching the Index:
  - You can search for:
    - Exact matches
      - "ear" matches only "ear", not "bear"
      - Autopsy default
    - Substrings
      - "ear" does match "bear"
    - Regular Expressions
      - Predefined regular expressions:
        - Phone (US)
        - IP Addresses
        - E-mail
        - URL
        - Credit Cards
      - Post processing Luhn validation is performed on credit card numbers
  - Terms can be grouped into lists to make it easier to manage and organize.
  - Two ways to specify words to search for:
    1. Specify known terms when running ingest on a new data source.
    2. Using the search box when in the main UI (ad-hoc)

Periodic Searching:
  - Keywords specified during ingest will be periodically searched for.
  - By default, Autopsy will save the index every 5 minutes and perform a search.
    - It will increase this duration as the index gets bigger.
  - This ensures you quickly get keyword hits on user content (which gets top priority for file scheduling).

Ad-hoc Searches:
  - Use the search box in the upper-right
  - Options for exact match, substring, and regular expression search.
  - Can restrict to certain data sources.
  - Can choose to not save results as artifacts
  - Results sent to new viewer.

Results located under the results tree in the Keyword Hits category.

***
## Correlation Engine Module

Two Features:
  1. Queries the Central Repository to see if items in the current case were seen before
  2. Adds data to the Central Repository about the current case.

Configuration:
  - What types to store
  - If to generate alerts

What is the Central Repository?
  - Database that stores data from past cases:
    - MD5 Hash Values
    - Comments
    - Wifi SSIDs
    - etc.
  - Allows examiners to easily access important data from past cases.
  - Autopsy typically has case-specific databases.

The module gets data from other modules (hash values, email addresses, SSIDs, etc.)
  - It queries the Central Repository to see if they previously occurred
  - It inserts the new data into the Central Repository.

Type of data that can be correlated:
  - MD5: from files
  - Domain: From web artifact URLs
  - Email Addresses: from email messages, keyword hits, contacts, etc.
  - Phone Numbers: From messages, contact books, call logs, etc.
  - USB Device IDs, From registry parsing
  - Wifi SSIDs: From registry parsing

What gets stored:
  - For each data type, the repository stores:
    - Value (hash, email, phone number, etc.)
    - Case
    - Data Source
    - File Path
    - User-supplied Comment
    - Notable Status
  - There is one row in the repository for every instance of a property.

Alerting:
  - If a property was previously seen, an alert may get generated:
    - Files will be flagged if it was previously tagged as "Notable"
    - USB Devices will always be flagged if they were previously seen.
***
## Android Analyzer Module
What it does:
  - Locates SQLite databases and files from Android and 3rd party apps.
  - Parses the databases
  - Adds results to the blackboard

Supported Inputs:
  - Must acquire first with another tool
  - Most physical acquisitions supported
    - Some older Android devices do not have a volume system -> Autopsy cannot find file systems on these devices
  - File system dumps (logical) should work.
  - Simply add the data as a data source.

Data Extracted:
  - Call Logs
  - Contacts
  - Messages
    - SMS / MMS
    - Tango
    - Words with Friends
    - Facebook Messenger
    - IMO
    - LINE
    - Skype
    - TextNow
    - Viber
    - WhatsApp
  - Browsers
    - Android browser
    - Opera
    - Samsung SBrowser
  - File Transfer
    - Sharelt
    - Xender
    - Zapya
  - Geo
    - Cache.wifi and cache.cell files
    - Browser and Google Maps
    - ORUX Maps
  - Android installed apps

Artifacts Created:
  - Call Logs
  - Messages
  - Contacts
  - Web history, cookies, bookmarks, and downloads
  - GPS points
  - Installed Apps

***
## Timeline Interface
What it does: Allows an examiner to graphically view activity on the system.
  - Developed with funding from DHS S&T

The Timeline is a visualization tool.
  - It relies on other modules to extract time stamps:
    - File times when data sources were added
    - Web activity from Recent Activity
    - Exif, Plaso, Android, etc.

Layout:
  - Filters: Upper-left
  - Events: Upper-right
  - Files and Content: bottom half

Three views:
  - Counts: Bar chart view to show how many events occurred
    - Default scale is not linear
    - Double click on each bar to change time scale to the range of that bar.
      - Start at years, can go to months, days, etc.
  - Details: shows specific events, clustered to prevent overload
    - Each cluster can be expanded or collapsed as needed.
    - Clusters can be pinned to keep track of them.
  - List: shows all events in order in a basic table

***
## Image Gallery
What it does:
  - Allows you to more easily review sets of images.
  - Developed with funding from DHS S&T

Displays images organized by folder.
  - No infinite scroll bar of thumbnails

Folders are displayed once contents have gone through the ingest modules.
  - Needs file type, hash, and Exif information

Folders are prioritized based on:
  - Density of hash hits
  - Number of images in folder

Layout:
  - Folder tree: upper left
  - Files Categorized: lower left
  - Current Group: Middle and right side

Image Borders: signify what is known about an image
  - Purple dashes are for hash hits
  - Other colors are for user tagging/categorization

Typical Usage:
  - Ingest disk image with hash databases, Exif, etc. enabled
  - Open Tools -> Image Gallery
  - Review first folder of files
    - Categorize or tag individual files.
    - Categorize entire group
  - Use tree on left to navigate around interesting folders.
  - Press "Next Unseen Group" button until all groups are reviewed.

Law Enforcement Bundle:
  - Free add-on from Basis Technology
  - Provides access to C4ALL and Project Vic databases.
  - Downloaded from http://www.autopsy.com

Project Vic Integration:
  - Populates Autopsy SQLite databases based on data in Project Vic Odata files.
  - Creates one SQLite databases per category.
  - Configuration:
    - Set a folder to store the SQLite databases
    - Import Odata files as Project Vic client is updated.

C4ALL:
  - Creates a new ingest module that looks hash values up in central database.
  - Autopsy does not provide database or data
  - Marks files as hash hits.

***
## Communications Interface
What it does:
  - Provides a more powerful way of viewing communication data.
    - It's oriented around accounts and not data types.
  - Examples:
    - Filters accounts by type and communication dates
    - Shows messages and data for a selected account.

Terminology:
  - **Account**: An address that people use to refer to someone / something. Accounts have a type (such as Email) and an identifier (such as jdoe@gmail.com).
  - **Relationship**: When two accounts communicate or know about each other. Examples:
    - Sending a message
    - Having an account ID in a contact book
  - **Device Account**: A special account created for each data source to represent the physical device when we don't have a better account ID.
    - Often used to define a relationship for call logs and contact books on a phone.
    - Example: Call log database defines only the other recipient.

What does it show?
  - Displays data extracted from other modules:
    - Android Analyzer
    - Email
  - It is account-oriented
    - You first pick an account and then see all activity associated with it.
  - It is relationship-based
    - Only accounts with a relationship are shown
    - Random email addresses found in a file are not shown.

***
## Tagging, Commenting, and Reporting
Tagging: Allows you to make a reference to a file or object and easily find it later.
  - Bookmarking
  - Allows you to comment on the file
  - Highlight a part of a picture that is relevant
  - Tag Names: create arbitrary tag names.
    - Autopsy remembers your names from previous cases
    - Make them by right clicking on a file
    - Specify if a tag is for notable items or not.
  - Tagging a Result:
    - Results (blackboard artifacts) are associated with a file
    - When you view a result, you have the choice to tag either:
      - The result
      - The file
    - The final report will focus on either the result or the file
  - Can be viewed in the Results Tree under Tags.

Comments: allow you to make notes about why you tagged an item.
  - Will be shown in reports
  - Can be saved in the Central Repository for future reference.
  - To comment on a file, choose "Tag and Comment"
  - Show up in table rows as a comment icon.
  - Comments are in Annotations viewer

Reporting: makes a report that you can send to others or use in other report formats.
  - It is an extensible framework
  - Comes with:
    - HTML and Excel Results (focuses on results and blackboard artifacts)
    - Text File (focuses on each file and metadata)
    - KML File (for Google Earth)
    - Add Tagged Files to Hash Set
    - Portable Case
  - HTML/Excel Reports
    - Document the analysis module results and tagged items
    - Two modes:
      - Report on all items
      - Report only on items with specific tags
  - You can specify sharing restrictions with headers and footers
  - Excel result report contains one data type per worksheet
  - File reports:
    - Show what files are in the case
    - One line per file
    - Columns for metadata
    - Can choose which columns
  - KML Report:
    - Generates KML file containing:
      - Exif Artifacts
      - GPS Trackpoints
      - GPS Route
    - Includes thumbnail of Exif images

Portable Case:
  - A portable Case is an Autopsy case that includes only a subset of the data from its original case:
    - Only the tagged files
    - Only the files with interesting item hits
  - It is self contained:
    - Has its own SQLite database
    - All files are located in the case folder
  - Can be shared with any users for review or assistance.

***
## Installing 3rd Party Modules
Autopsy was written to be a platform for plug-in modules
  - Modules are written in Java or Python

Github repository exits for publicly available modules: http://github.com/sleuthkit/autopsy_addon_modules

Module Concepts:
  - Officially, modules may only work on a given major version number of Autopsy.
    - Module written for 3.0.8 should worth with 3.0.9, but not 3.1.0
  - In practice, we don't break backward compatibility. Modules should keep on working.

Java Modules:
  - Java modules will have a `.nbm` extension.
  - `.nbm` files are NetBean Module files
    - They contain several Autopsy modules
    - NetBeans allows these modules to be auto-updated and downloaded
  - These modules can verify they are being installed into the correct version of Autopsy.

Install Java Module:
  - Tools -> Plugins menu
  - Choose Downloaded Tab
  - Choose "Add Plugins..."
  - Once added, click Install

Python Modules:
  - Python Modules are a folder that contain one or more `.py` files.
  - Python modules do not verify versions
  - All Python modules need to be copied as sub-folders in a central location.
  - Only ingest and report modules can be written in Python.

Install Python Module:
  - Tools -> Python Plugins
  - Copy the module folder into the plugins folder.

***
