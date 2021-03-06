Derived data
============

Derived data include things like:

- coordinates of important points in an image
- sum of intensities from regions of interest
- number of coordinates in regions of interest

For example to segment using the watershed algorithm we need to find local
minima.

.. code-block:: python

    >>> from jicimagelib.calculate import LocalMinima
    >>> from jicimagelib.transform import Watershed
    >>> local_minima = LocalMinima(org_im)
    >>> segmented_im = Watershed(org_im, local_minima)

To visualise what is going on one may use something along the below:

.. code-block:: python

    >>> from jicimagelib.viewer import ImageViwer
    >>> from jicimagelib.render import Renderer
    >>> viewer = ImageViwer()
    >>> renderer = Renderer()
    >>> circles = renderer.draw_circles(local_minima)
    >>> viewer.load(segmented_im)
    >>> viewer.add_image_layer(circles)

.. note:: In the above local_minima is probably an instance of a class that
          contains both an image and coordinates of the local minima.

Now we may want to calculate the sum of the intensities in the segmented
regions of interest.

.. code-block:: python

    >>> from jicimagelib.calculate import ROIProperty
    >>> rois_sum_intensities = ROIProperty(roi=segmented_im, input=org_im, func=sum)

Or the number of coordinates in each region of interest.

.. code-block:: python

    >>> coords_in_rois = ROIProperty(roi=segmented_im, input=local_minima, func=count_coords_in_roi)

.. note:: The :func:`count_coords_in_roi` is a custom built function that takes
          as input a segmented image and a set of coordinates.
