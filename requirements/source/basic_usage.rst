Basic usage
===========

To load and work with microscopy data we use an :class:`ImageDataManager`. This has
the advantage that it can load microscopy files and make them available to the
user as augmented ``numpy`` arrays.

In the simplest instance an :class:`ImageDataManager` makes use of the BioFormats
``bfconvert`` tool to convert the microscopy file into a series of tiff files
stored in a directory. Note that this is an implementation detail and the user
can seamlessly access the augmented ``numpy`` arrays to work with data.

.. code-block:: python

    >>> from jicimagelib import ImageDataManager
    >>> image_data_manager = ImageDataManager()

By default the :class:`ImageDataManager` simply uses a directory as a backend and
points to the temporary directory ``/tmp/jicimagelib/``.


.. code-block:: python

    >>> image_data_manager.backend_type
    'directory'
    >>> image_data_manager.backend_location
    '/tmp/jicimagelib/'
   
To load a microscopy file into the :class:`ImageDataManager` we use the
:func:`ImageDataManager.load` function, which returns the identifier of the
loaded object.

.. code-block:: python

    >>> image_data_manager.load('test1.lif')
    0
    >>> image_data_manager.load('test2.lif')
    1

.. note:: The same microscopy file can be added several times, and each time it
          will be given unique idetifiers. For example loading the ``test1.lif``
          file again will create another entry in the ``ImageDataManager``
          instance.

          .. code-block:: python

             >>> image_data_manager.load('test1.lif')
             2

We can access the microscopy entries using the
:attr:`ImageDataManager.entries` attribute.

.. code-block:: python

    >>> image_data_manager.entries
    [<MicroscopyEntry 0>, <MicroscopyEntry 1>, <MicroscopyEntry 2>]
    >>> first_entry = image_data_manager.entries[0]

A :class:`MicroscopyEntry` has several attributes including:

- :attr:`MicroscopyEntry.identifier`
- :attr:`MicroscopyEntry.name`
- :attr:`MicroscopyEntry.channels`
- :attr:`MicroscopyEntry.z_slices`
- :attr:`MicroscopyEntry.time_points`


.. code-block:: python

    >>> first_entry.identifier
    0
    >>> first_entry.name
    'test1.lif1'
    >>> first_entry.channels
    [<Channel 0>, <Channel 1>, <Channel 2>]
    >>> first_entry.z_slices
    [<ZSlice 0>, <ZSlice 1>, ..., <ZSlice 20>]
    >>> first_entry.time_points
    [<TimePoint 0>, <TimePoint 1>, ..., <TimePoint 10>]

The :class:`MicroscopyEntry`, :class:`Channel`, :class:`ZSlice`,
:class:`TimePoint` all have a :func:`get_image` function which takes arguments
to specify the ``channel``, ``z_slice`` and ``time_point``.

So suppose one wanted to work on an :class:`Image` in channel 0, z-slice 3,
time point 5 one could use any of the below.


.. code-block:: python
    
   >>> first_entry.channels[0].z_slices[3].time_points[5].get_image()
   <Image MicroscopyEntry(0) Channel(0) ZSlice(3) TimePoint(5)>
   >>> first_entry.channels[0].z_slices[3].get_image(time_point=5)
   <Image MicroscopyEntry(0) Channel(0) ZSlice(3) TimePoint(5)>
   >>> first_entry.channels[0].get_image(time_point=5, z_slice=3)
   <Image MicroscopyEntry(0) Channel(0) ZSlice(3) TimePoint(5)>
   >>> im = first_entry.get_image(channel=0, z_slice=3, TimePoint(5)>
   <Image MicroscopyEntry(0) Channel(0) ZSlice(3) TimePoint(5)>

This can be achieved because each instance of a :class:`Channel`,
:class:`ZSlice`, :class:`TimePoint` knows where it came from.

.. code-block:: python
    
    >>> channel0 = first_entry.channels[0]
    >>> channel0.channel
    0
    >>> channel0.z_slice
    None
    >>> channel0.time_point
    None
    >>> z_slice3 = channel0.z_slices[3]
    >>> z_slice3.channel
    0
    >>> z_slice3.z_slice
    3
    >>> z_slice3.time_point
    None
    >>> time_point5 = z_slice3.time_points[5]
    >>> time_point5.channel
    0
    >>> time_point5.z_slice
    3
    >>> time_point5.time_point
    5
    
So suppose you wanted to loop over all the z-slices in channel 2 at time point
9 you could achieve using the code snippet below.

.. code-block:: python

    >>> for z_slice in first_entry.channels[2].time_points[9].z_slices:
    ...     im = z_slice.get_image()
    ...

Alternatively, one could use the code snippet below.

.. code-block:: python

    >>> for z_slice in first_entry.z_slices:
    ...     im = z_slice.get_image(channel=2, time_point=9)
    ...