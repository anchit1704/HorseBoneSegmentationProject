# HorseBoneSegmentationProject

In this project, I develop a threshold based image segmentation algorithm to detect bone vs non bone regions in a CT scan. 

Steps - 

1. Annotate the CT images using 3D slicer tool. You can download 3D slicer from [here](https://download.slicer.org/). The original and segmented images are stored in the horseName.nrrd and horseName.seg.nrrd  in the 3D slicer output folder respectively.
2. Reading the data -
     ```
     import nrrd
     import slicerio
     data,k = nrrd.read(path + "Tizadaisy LF.nrrd")
     voxels, header = nrrd.read(path + "Tizadaisy LF Segmentation.seg.nrrd")
     segmentation_info = slicerio.read_segmentation_info(path + "Tizadaisy LF Segmentation.seg.nrrd")
     segment_names_to_labels = [("LF Limb", 10), ("LF MC3", 12), ("LF MC2", 6), ("LF MC4", 8), ("Medial PSB", 4), ("Lateral PSB", 16), ("LF P1", 9)]
     extracted_voxels, extracted_header = slicerio.extract_segments(voxels, header, segmentation_info, segment_names_to_labels)
     ```
3. Plotting the data -

   Single slice -
   ```
   plt.imshow(data.T[80], cmap='gray')
   plt.show()
   ```
     
   ![data](https://github.com/anchit1704/HorseBoneSegmentationProject/assets/17884278/e75f58af-9f79-46db-8154-d6ba5e5af169)
