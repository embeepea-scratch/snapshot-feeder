This project contains the script `feeder` which generates and/or
updates a set of CSV _feed_ files that enables the Drupal Data Snapshots
module to import metadata associated with an archive of images.

Overview
--------

The Data Snapshots module provides a uniform way to view
images associated with various climate data sets.  The module
defines two custom content types: _Data Snapshot Data Source_,
and _Data Snapshot_.

Nodes of the _Data Snapshot Data Source_ content type correspond to
distinct sources of data.  Each Data Source corresponds to a specific
data set which issues new data releases on a regular basis.  For example,
the "Total Monthly Precipitation" data source issues a new data release
each month, and the "Drought Monitor" data source issues a new data
release each week.  There will be a small number of nodes of type
_Data Snapshot Data Source_; each one must be entered/edited manually,
and its content generally does not change very often.

Nodes of the _Data Snapshot_ content type correspond to data associated
with a specific date for one _Data Source_.  Each _Data Snapshot_ node 
stores the URLs to several images representing the
data from that data source at that date.  For example, there is one
_Data Snapshot_ node corresponding to each month's data release for
the "Total Monthly Precipitation" data source.  Each of these nodes
contains links to 5 different images at various sizes/resolutions.

In general there are many nodes of type _Data Snapshot_.
For each data source, there is a _Data Snapshot_ node corresponding
to each date for which there are images from that data source.
These nodes are typically not entered manually, rather they are imported
in batches either automatically, or via the Drupal administrative
interface.

Note that the _Data Snapshot_ nodes do not store the images themselves --
only the URLS of them; the images are expected to be hosted on
an image server that is external to the Drupal site.

This file documents the file and directory structure that must be
present on the image server, and the `feeder` script which generates
and/or updates a set of feed files on the image server that the Data
Snapshos module uses to import new batches of _Data Snapshot_ nodes.

Note: In older versions of the Data Snapshots module the content type
_Data Snapshot Data Source_ was called _Data Snapshot Data Set_, and
there are still a few places in the module code and comments where the
term _data set_ is used instead of _data source_.  So in general, when
reading code and developer documentation associated with Data
Snapshots, the terms "data set" and "data source" should be considered
to be synonomous.

Image Server Directory Structure
--------------------------------

Images on the image server should be stored in a directory heirarchy
as follows.  At the top level is a single directory. The name and
location of this directory doesn't matter; we'll refer to it as
`IMAGE_ROOT`.  The `IMAGE_ROOT` directory contains one _data source image
directory_ for each distinct _Data Source_, and each data source image
directory contains one _resolution directory_ for each size/resolution
of images available for that data source.

Each data source image directory is named by the _machine name_ of a
data source.  (Each _Data Source_ has a field called _Machine Name_
which specifies a machine-friendly string that is used to uniquely
identify that data source.)

The resolution directories have names like `1000`, `620`, `diy`, `hd`,
`hdsd`.  The images themselves are stored in the resolution
directories.

The names of the image files follow the very specific pattern:

```
    [machine-name]-[resolution]-[YYYY-MM-DD].*
```

For example:

```
  IMAGE_ROOT/
    totalprecip-monthly-cmb/                   <== data source image directory
      1000/                                    <== resolution directory
        totalprecip-monthly-cmb--1000x690--2000-01-00.png
        totalprecip-monthly-cmb--1000x690--2000-02-00.png
        ...
      620/                                     <== resolution directory
        totalprecip-monthly-cmb--620x450--2000-01-00.png
        totalprecip-monthly-cmb--620x450--2000-02-00.png
        ...
      diy/                                     <== resolution directory
        totalprecip-monthly-cmb--4096x2623--2000-01-00.zip
        totalprecip-monthly-cmb--4096x2623--2000-02-00.zip
        ...
      hd/                                      <== resolution directory
        totalprecip-monthly-cmb--1920x1080hd--2000-01-00.png
        totalprecip-monthly-cmb--1920x1080hd--2000-21-00.png
        ...
      hdsd/                                    <== resolution directory
        totalprecip-monthly-cmb--1920x1080hdsd--2000-01-00.png
        totalprecip-monthly-cmb--1920x1080hdsd--2000-02-00.png
        ...
      web/                                     <== web resolution symlink directory
        totalprecip-monthly-cmb--web--2000-01-00.png
        totalprecip-monthly-cmb--web--2000-02-00.png
        ...
    usdroughtmonitor-weekly-ndmc/              <== data source image directory
      1000/                                    <== resolution directory
        usdroughtmonitor-weekly-ndmc--1000x704--2010-01-05.png
        usdroughtmonitor-weekly-ndmc--1000x704--2010-01-12.png
        ...
      620/                                     <== resolution directory
        usdroughtmonitor-weekly-ndmc--620x464--2010-01-05.png
        usdroughtmonitor-weekly-ndmc--620x464--2010-01-12.png
        ...
      diy/                                     <== resolution directory
        usdroughtmonitor-weekly-ndmc--4096x2048--2010-01-05.zip
        usdroughtmonitor-weekly-ndmc--4096x2048--2010-01-12.zip
        ...
      hd/                                      <== resolution directory
        usdroughtmonitor-weekly-ndmc--1920x1080hd--2010-01-05.png
        usdroughtmonitor-weekly-ndmc--1920x1080hd--2010-01-12.png
        ...
      hdsd/                                    <== resolution directory
        usdroughtmonitor-weekly-ndmc--1920x1080hdsd--2010-01-05.png
        usdroughtmonitor-weekly-ndmc--1920x1080hdsd--2010-01-12.png
        ...
      web/                                     <== web resolution symlink directory
        usdroughtmonitor-weekly-ndmc--web--2010-01-05.png
        usdroughtmonitor-weekly-ndmc--web--2010-01-12.png
    ...
```

Web Resolution Directory Symlinks
---------------------------------

In addition to the `620`,`1000`,`diy`,`hd`, and `hdsd` directories, 
each data source image directory contains a directory named `web`.
This directory does not
contain images but rather contains symlinks to images in the `620`
directory.  These symlinks are used in contexts where
a URL with a predictable pattern is needed to refer to an image
suitable for displaying in a web page -- the names of these symlinks
depend only on the data set name and the date, and omit the exact
pixel dimensions of the image.  (The heights of the `620` resolution
images are not the same for all data sets, so the names of the
`620` size image files can't easily be created based just on the
data source name and date.)

These symlinks are created and maintained by the feeder script.  In general,
`feeder` creates a `web` symlink for an image whenever it adds that image to
a snapshot feed file.  You can also use the `--weblinks-only` option to force
it to recreate all web links for all images.


Feed Files
----------

The Data Snapshots module imports image URLS and metadata from the image
server through a collection of _feed_ files.  There are two types of feed files:
_master feed files_, and _snapshot feed files_.

The _snapshot feed files_ contain URLs and other data associated with
_Data Snapshot_ nodes; each line in one of these files contains the information
that Drupal uses to import one _Data Snapshot_ node.  These
files are named according to one of the patterns

  ```
      [machine-name]--[YYYY-MM-DD--HH-mm-ss].csv
      [machine-name]--[YYYY-MM-DD--HH-mm-ss].NNN.csv
  ```

where `[machine-name]` is the machine name of the data source, `[YYYY-MM-DD-HH-mm-ss]` is
a time stamp that indicates when the feed file was generated, and `NNN` is a sequence
number (not all snapshot feed files have sequence numbers).  The time stamp does
not necessarily relate to the dates of the images mentioned in the file -- it is simply
used as a unique key to make it easy to keep track of whether the file has been downloaded and
processed.
  
The intention for these snapshot feed files is that whenever new images are added
for a data source, a new snapshot feed file is created for that data source, containing
the image URLS and metadata for the _Data Snapshot_ nodes corresponding to those new
images.

In general, there will be a collection of snapshot feed files corresponding
to each data source, and whenever new images are added for the data source,
a new snapshot feed file is generated for those images.

In addition to a collection of snapshot feed files, there is also a
single _maser feed file_ associated with each data source (one master
feed file for each data source).  The master feed file simply contains
the URLs of all the snapshot feed files for the data source.  The
master feed file is named `feed.csv` and is located in the same
directory where that data source's snapshot feed files live.


The Import Process
------------------

On the Data Snapshots module settings page in the Drupal
administrative user interface, there is a button labeled _Run Import
Now_.  Clicking that button will cause Drupal to download the master
feed files for all the current Data Source nodes, and to make a list
of all the snapshot feed file URLs that they contain.  It then
eliminates from this list any URLs that it has previously processed,
and for each URL that remains, the snapshot feed file is downloaded
and Data Snapshot nodes are created for each line in the file.  The
URL of the snapshot feed file is then added to Drupal's internal list
of snapshot feed file URLs that have been processed, so that the same
file will not be downloaded again in the future.

NOTE: The list of URLs of snapshot feed files that have been processed
is stored in the custom table `data_snapshots_import_urls`.  There is
no way to clear out this table from the GUI; if you want to clear this
table to re-import all Data Snapshot nodes, you'll have to do so by
hand.  Note that the import process does NOT check for duplicate
nodes, though, so it's important to delete all nodes of type Data
Snapshot before doing this.

Feed Generation and Updating
----------------------------

The `feeder` script examines all the images in the entire IMAGE_ROOT
directory heirarchy, and generates or updates the snapshot feed files
and master feed files accordingly. `Feeder` also creates and/or updates
the `web` directory symlinks; see _Web Resolution Directory Symlinks_ above
for more details about that.

By default, when invoked with no arguments, `feeder` examines both
the existing images and the existing snapshot feed files present for
each data source.  If there are any images present that are not
included in any existing snapshot feed files for the data source, it
creates a new snapshot feed file for those images.  (No new snapshot
feed file is created for a data source if all the images for it are
already contained in existing snapshot feed files.)  For each new
snapshot feed file it creates, `feeder` updates the corresponding master
feed files by appending the URL of the new snapshot feed file to the
end.

So, the normal workflow for adding images to the image server
and importing new Data Snapshot nodes into the site is as follows:

  1. Add new batches of images to the appropriate directories
     under `IMAGE_ROOT` on the image server.  It does not really matter
     how many images are added, or whether images are added to all
     of the data sources, or just some.  It _is_ important, though,
     that a complete set of resolutions be added for each date
     for which images are added to a data source.
  2. Run the _feeder_ script on the image server to update the feed files.
  3. Click the _Run Import Now_ button on the Data Snapshots module
     settings page in the Drupal administrative user interface to
     import the new feed files and create the corresponding Data Snapshot
     nodes.

`feeder` can also be run with varous command-line arguments
which cause it to behave differently, for example to regenerate new
feed files for all images, or to limit the number of images included
in any one snapshot feed file.  Run `feeder --help`, or see the source
code and comments, for details.
