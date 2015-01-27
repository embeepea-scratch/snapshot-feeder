This project contains the script `feeder` which generates and/or
updates a set of CSV _feed_ files that allow the Drupal Data Snapshots
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
release each week.  The site has a small number of nodes of this type;
each one must be entered/edited manually, and its content generally does
not change very often.

Nodes of the _Data Snapshot_ content type correspond to data associated
with a specific date for one _Data Source_.  Each _Data Snapshot_ node 
stores links to several images representing the
data from that data source at that date.  For example, there is one
_Data Snapshot_ node corresponding to each month's data release for
the "Total Monthly Precipitation" data source.  Each of these nodes
contains links to 5 different images at various sizes/resolutions.

In general there are many nodes of type _Data Snapshot_ in the site:
for each data source, there is a _Data Snapshot_ node corresponding
to each date for which there are images from that data source.
These nodes are typically not entered manually, rather they are imported
in batches either automatically, or via the Drupal administrative
interface.

Note that the _Data Snapshot_ nodes do not store the images themselves --
only links to them; the images are expected to be hosted on
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
    [machine-name]-[resolution]-YYYY-MM-DD.*
```

In addition to the resolution directories that contain the images
themselves, each data source image directory also contains one additional
resolution directory named `web`.  This directory does not contain images
but rather contains symlinks to images in one of the other resolution directories
These symlinks are used in contexts where a URL
with a predictable pattern is needed to point to an image suitable for
displaying in a web page.  These symlinks typically refer to the images
in the `620` resolution directory, but the names of the `web` symlinks can
be easily constructed from the data source name and date, as opposed
to the names of the `620` resolution images, which include the image height
which is not the same for every data set.

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
