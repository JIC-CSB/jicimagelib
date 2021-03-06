Quick start guide
=================


Introdcution
------------

The :mod:`jicimagelib` package has been designed to make it easy to work with
microscopy images. To illustrate its use we will use a
`multi-channel-4D-series.ome.tif
<http://www.openmicroscopy.org/Schemas/Samples/2015-01/bioformats-artificial/multi-channel-4D-series.ome.tif.zip>`_
file.

.. code-block:: python

    >>> fpath = "multi-channel-4D-series.ome.tif"

Let us also define a directory for caching individual files from any microscopy
data that we may load.

.. code-block:: python

    >>> backend_directory = "./backend"


..
    This is just to make the doctest pass.

    >>> import os.path
    >>> import jicimagelib
    >>> JICIMAGLIB = os.path.dirname(jicimagelib.__file__)
    >>> fpath = os.path.join(JICIMAGLIB, "..", "tests", "data", fpath)

Loading a microscopy file
-------------------------

Let us set up a :class:`jicimagelib.image.DataManager`. To instantiate a
:class:`jicimagelib.image.DataManager` we need a backend.  Let us make use of a
:class:`jicimagelib.io.FileBackend`.

.. code-block:: python

    >>> from jicimagelib.image import DataManager
    >>> from jicimagelib.io import FileBackend
    >>> backend = FileBackend(backend_directory)
    >>> data_manager = DataManager(backend)

Now let us load the microscopy file into the ``data_manager``.

.. code-block:: python

    >>> collection = data_manager.load(fpath)


Accessing individual images
---------------------------

The data manager is essentially a list containing
:class:`jicimagelib.image.ImageCollection` or
:class:`jicimagelib.image.MicroscopyCollection` instances.

.. code-block:: python

    >>> len(data_manager)
    1

And a :class:`jicimagelib.image.ImageCollection` is essentially just a list of
:class:`jicimagelib.image.MicroscopyImage` instances.

.. code-block:: python

    >>> len(collection)
    105
    >>> collection  # doctest: +ELLIPSIS, +NORMALIZE_WHITESPACE
    [<MicroscopyImage(s=0, c=0, z=0, t=0) object at ...>,
     <MicroscopyImage(s=0, c=0, z=0, t=1) object at ...>,
      ...,
     <MicroscopyImage(s=0, c=2, z=4, t=6) object at ...>]

The :class:`jicimagelib.image.MicroscopyImage` contains meta data about the
microscopy image.

.. code-block:: python

    >>> microscopy_image = collection[0]
    >>> microscopy_image.series
    0
    >>> microscopy_image.channel
    0
    >>> microscopy_image.zslice
    0
    >>> microscopy_image.timepoint
    0

One can use this meta data to access a specific
:class:`jicimagelib.image.MicroscopyImage` using the
:func:`jicimagelib.image.ImageCollection.proxy_image` function.

.. code-block:: python

    >>> collection.proxy_image(s=0, c=1, z=2, t=3)  # doctest: +ELLIPSIS
    <MicroscopyImage(s=0, c=1, z=2, t=3) object at ...>


One can access the raw 2D :class:`jicimagelib.image.Image` instance
from the :attr:`jicimage.image.MicroscopyImage.image` attribute.

.. code-block:: python

    >>> microscopy_image.image  # doctest: +ELLIPSIS, +NORMALIZE_WHITESPACE
    Image([[ 0,  0,  0, ...,  0,  0,  0],
           [ 1,  1,  1, ...,  1,  1,  1],
           [ 2,  2,  2, ...,  2,  2,  2],
           ..., 
           [95, 95, 95, ..., 95, 95, 95],
           [95, 95, 95, ..., 95, 95, 95],
           [96, 96, 96, ..., 96, 96, 96]], dtype=uint8)

.. note:: The :class:`jicimagelib.image.Image` class is a subclass of
          :class:`numpy.ndarray`.

It is also possible to access the raw 2D :class:`jicimagelib.image.Image`
instance from the :class:`jicimagelib.image.ImageCollection` directly,
side-stepping the :class:`jicimagelib.image.MicroscopyImage`, using the
:func:`jicimagelib.image.ImageCollection.image` funciton.

.. code-block:: python

    >>> image = collection.image(s=0, c=1, z=2, t=3)
    >>> image  # doctest: +ELLIPSIS
    Image([[ 0,  0,  0, ...,  0,  0,  0],
           [ 1,  1,  1, ...,  1,  1,  1],
           [ 2,  2,  2, ...,  2,  2,  2],
           ..., 
           [95, 95, 95, ..., 95, 95, 95],
           [95, 95, 95, ..., 95, 95, 95],
           [96, 96, 96, ..., 96, 96, 96]], dtype=uint8)



Working with transformations
----------------------------

Suppose that we wanted to create a transformation to invert our image. We can
achieve this by importing the :func:`jicimagelib.transform.transformation`
decorator.

.. code-block:: python

    >>> import numpy as np
    >>> from jicimagelib.transform import transformation
    >>> @transformation
    ... def invert(image):
    ...     """Return an inverted image."""
    ...     maximum = np.iinfo(image.dtype).max
    ...     maximum_array = np.ones(image.shape, dtype=image.dtype) * maximum
    ...     return maximum_array - image
    ...

..
    # We do not want to write out the transforms to disk.
    >>> from jicimagelib.io import AutoWrite
    >>> AutoWrite.on = False

We can now apply the transformation to our image.

.. code-block:: python

    >>> inverted_image = invert(image)
    >>> inverted_image
    Image([[255, 255, 255, ..., 255, 255, 255],
           [254, 254, 254, ..., 254, 254, 254],
           [253, 253, 253, ..., 253, 253, 253],
           ..., 
           [160, 160, 160, ..., 160, 160, 160],
           [160, 160, 160, ..., 160, 160, 160],
           [159, 159, 159, ..., 159, 159, 159]], dtype=uint8)

Understanding the history of an image
-------------------------------------

When working interactively with images it can be useful to understand where an
image originally came from and what transformations it has undergone. This
information is avaialable in the :attr:`jicimagelib.image.Image.history`
attribute.

Let us have a look at the history of our ``inverted_image``.

.. code-block:: python

    >>> inverted_image.history  # doctest: +ELLIPSIS, +NORMALIZE_WHITESPACE
    ['Created image from .../multi-channel-4D-series_S0_C1_Z2_T3.tif',
     'Applied invert transform']

..
    Tidy up: remove the ./backend directory we created.

    >>> import shutil
    >>> shutil.rmtree(backend_directory)
