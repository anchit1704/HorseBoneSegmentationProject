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

   CT image plot -
   
   ```
   plt.imshow(data.T[80], cmap='gray')
   plt.show()
   ```
   ![data](https://github.com/anchit1704/HorseBoneSegmentationProject/assets/17884278/b13ce4a2-5b9e-4590-b90d-e35fca6dcdc7)

   Ground truth Segmentation plot -

   ```
    extracted_voxels.T[80][340:,:] = 0
    plt.imshow(extracted_voxels.T[80], cmap='gray')
    plt.show()
   ```
   
   ![seg](https://github.com/anchit1704/HorseBoneSegmentationProject/assets/17884278/5145471a-dc2a-4454-817e-527fba97381c)

4. Create segmentation using pixel threshold value -

   ```
   import cv2 
   for i in range(228):
       print(i)
       gray = data.T[i].copy()
       gray_r = gray.reshape(gray.shape[0]*gray.shape[1])
       thresh = 550
       for j in range(gray_r.shape[0]):
           if gray_r[j] > thresh and j < 174080:
               gray_r[j] = 1
           else:
               gray_r[j] = 0
   
       gray = gray_r.reshape(gray.shape[0],gray.shape[1])
       plt.imshow(gray, cmap='gray')

       plt.show()
    ```
    ![threshold](https://github.com/anchit1704/HorseBoneSegmentationProject/assets/17884278/9011d7b8-0aa3-4412-9827-790fe764cda2)
    
   
