# HorseBoneSegmentationProject

In this project, I develop a threshold based image segmentation algorithm to detect bone vs non bone regions in a CT scan. 

Steps - 

1. Annotate the CT images using 3D slicer tool. You can download 3D slicer from [here](https://download.slicer.org/). The original and segmented images are stored in the horseName.nrrd and horseName.seg.nrrd  in the 3D slicer output folder respectively.
2. Reading the data -
     ''' python

     import nrrd
     import slicerio
     data,k = nrrd.read(path + "Tizadaisy LF.nrrd")
     voxels, header = nrrd.read(path + "Tizadaisy LF Segmentation.seg.nrrd")
     segmentation_info = slicerio.read_segmentation_info(path + "Tizadaisy LF Segmentation.seg.nrrd")
     segment_names_to_labels = [("LF Limb", 10), ("LF MC3", 12), ("LF MC2", 6), ("LF MC4", 8), ("Medial PSB", 4), ("Lateral PSB", 16), ("LF P1", 9)] '''
     
