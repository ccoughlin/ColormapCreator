ColormapCreator - matplotlib colormap creator
=============================================

ColormapCreator is a simple tool to create custom [matplotlib](http://matplotlib.org) colormaps.

How To Use
----------
Fire up the user interface, start adding colors as R,G,B values between 0-1 and separated by commas (the UI starts up with a colormap so you can see how it's done).  Edit, reorder, delete colors as desired.  Two types of colormap are supported - a linear colormap creates a gradient of your colors by linearly interpolating between them, and a list simply uses your colors as-is with no interpolation.  Click the Preview button to redraw the image plot with your colormap.

Colormaps are saved as text files in JSON format.  The easiest way to use them in your app is to write a wrapper for the standard matplotlib.cm.get_cmap() function like so:

```python
import matplotlib.cm as cm
import matplotlib.colors as colors
import os
import os.path

def colormaps_path():
    """Returns application's default path for storing user-defined colormaps"""
    return os.path.dirname(__file__)

def get_system_colormaps():
    """Returns the list of colormaps that ship with matplotlib"""
    return [m for m in cm.datad]

def get_user_colormaps(cmap_fldr=colormaps_path()):
    """Returns a list of user-defined colormaps in the specified folder (defaults to
    standard colormaps folder if not specified)."""
    user_colormaps = []
    for root, dirs, files in os.walk(cmap_fldr):
        for name in files:
            with open(os.path.join(root, name), "r") as fidin:
                cmap_dict = json.load(fidin)
                user_colormaps.append(cmap_dict.get('name', name))
    return user_colormaps

def load_colormap(json_file):
    """Generates and returns a matplotlib colormap from the specified JSON file,
    or None if the file was invalid."""
    colormap = None
    with open(json_file, "r") as fidin:
        cmap_dict = json.load(fidin)
        if cmap_dict.get('colors', None) is None:
            return colormap
        colormap_type = cmap_dict.get('type', 'linear')
        colormap_name = cmap_dict.get('name', os.path.basename(json_file))
        if colormap_type == 'linear':
            colormap = colors.LinearSegmentedColormap.from_list(name=colormap_name,
                                                                           colors=cmap_dict['colors'])
        elif colormap_type == 'list':
            colormap = colors.ListedColormap(name=colormap_name, colors=cmap_dict['colors'])
    return colormap

def get_cmap(cmap_name, cmap_folder=colormaps_path()):
    """Returns the matplotlib colormap of the specified name - if not found in the predefined
    colormaps, searches for the colormap in the specified folder (defaults to standard colormaps
    folder if not specified)."""
    user_colormaps = get_user_colormaps(cmap_folder)
    system_colormaps = get_system_colormaps()
    if cmap_name in system_colormaps:
        return cm.get_cmap(cmap_name)
    elif cmap_name in user_colormaps:
        cmap_file = os.path.join(cmap_folder, cmap_name)
        cmap = load_colormap(cmap_file)
    return cmap
```

Requirements
------------
Python, matplotlib, wxPython, SciPy, NumPy.

Author
------
[Chris Coughlin](http://www.chriscoughlin.com)