.. py:currentmodule:: bob.rppg.base


CHROM user's guide
==================

This package contains a free-software implementation of the Technical University of Eindhoven's
IEEE Transactions on Biomedical Engineering article "Robust Pulse Rate from Chrominance based rPPG", 
[dehaan-tbe-2013]_::

  @article{deHaan:2013,
    author={G. de Haan and V. Jeanne},
    journal={IEEE Transactions on Biomedical Engineering},
    title={Robust Pulse Rate From Chrominance-Based rPPG},
    year={2013},
    volume={60},
    number={10},
    pages={2878-2886}, 
  }

The algorithm to retrieve the pulse signal can be divided into several steps:

  1. Extracting skin color signals from the video sequence.
  2. Projecting the mean skin color in the defined chrominance space.
  3. Bandpass filtering in the chrominance space.
  4. Building the pulse signal.


Extract the pulse signal from video sequence 
--------------------------------------------

This scripts first load bounding boxes containing the faces in each frame
of each video sequence (if available) 
Otherwise, it will run a face detector (using :py:func:`bob.ip.facedetect.detect_single_face`).
Then, a skin color filter (:py:mod:`bob.ip.skincolorfilter`)
is applied to retrieve a mask containing skin pixels.

The mean skin color value is then computed, and projected onto the XY chrominance
colorspace. The signals in this colorspace are filtered using a bandpass filter
before the final pulse signal is built.

To extract the pulse signal from video sequences, do the following::

  $ ./bin/bob_rppg_chrom_pulse.py config.py -vv

To see the full options, including parameters and protocols, type:: 

  $ ./bin/bob_rppg_chrom_pulse.py --help 

As you can see, the script takes a configuration file as argument. This
configuration file is required to at least specify the database, but can also
be used to provide various parameters. A full example of configuration is
given below.

.. code-block:: python

  import os, sys

  # DATABASE
  import bob.db.hci_tagging
  import bob.db.hci_tagging.driver
  if os.path.isdir(bob.db.hci_tagging.driver.DATABASE_LOCATION):
    dbdir = bob.db.hci_tagging.driver.DATABASE_LOCATION
  if dbdir == '':
    print("You should provide a directory where the DB is located")
    sys.exit()
  database = bob.db.hci_tagging.Database()
  protocol = 'cvpr14'
  framerate = 61

  basedir = 'chrom-hci-cvpr14/'

  # EXTRACT PULSE 
  pulsedir = basedir + 'pulse'
  start = 306
  end = 2136
  motion = 0
  threshold = 0.1
  skininit = True
  order = 128
  window = 0

  # FREQUENCY ANALYSIS
  hrdir = basedir + 'hr'
  nsegments = 12
  nfft = 2048

  # RESULTS
  resultdir = basedir + 'results'


.. note::

   The execution of this script is very slow - mainly due to the face detection. 
   You can speed it up using the gridtk_ (especially, if you're at Idiap). For example::

     $ ./bin/jman sub -t 3490 -- ./bin/bob_rppg_chrom_pulse.py config.py

   The number of jobs (i.e. 3490) is given by typing::
     
     $ ./bin/bob_rppg_chrom_pulse.py config.py --gridcount


.. _gridtk: https://pypi.python.org/pypi/gridtk
