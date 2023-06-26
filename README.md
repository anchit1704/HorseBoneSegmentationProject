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
    
    ![thresh](https://github.com/anchit1704/HorseBoneSegmentationProject/assets/17884278/1e930f82-5af9-4692-aae7-ea01b98797e5)

5. Visualizing the results and measuring the performance

   ```
   # Image registration with green(ground truth) and red images(threshold based)

   import numpy as np
   import matplotlib.pyplot as plt
   import cv2
   import pandas as pd
   #for loop for iterating every slides
   metrics_list = []
   for k in range(45, 200):
       # create green images from 3D slicer segmentation images
       new_input =np.zeros((512,512,3))
       for i in range(extracted_voxels.T[k,:,:].shape[0]):
           for j in range(extracted_voxels.T[k,:,:].shape[1]):
               if extracted_voxels.T[k,:,:][i,j] != 0 and j>250:
                   new_input[i,j,1] = 255
                     
       #Obtain threshold based images            
       gray = data.T[k].copy()
       gray_r = gray.reshape(gray.shape[0]*gray.shape[1])
       thresh = 550
       for j in range(gray_r.shape[0]):
           if gray_r[j] > thresh and j < 174080:
               gray_r[j] = 1
           else:
               gray_r[j] = 0
   
       gray = gray_r.reshape(gray.shape[0],gray.shape[1])
       plt.imshow(gray, cmap='gray')
       print(k)
       #plt.savefig('E:\\Downloads\\DowloadsLatest\\UKY\\Research\\HorseBoneSegmentation\\Results\\Threshold\\Tizadaisy LF\\'+'Slice_'+str(k))
       plt.show()
       
       img = gray.copy()
       edge_image = cv2.Canny(img.astype(np.uint8),0,1)
       #showing Edged image
       plt.imshow(edge_image, cmap='gray')
       plt.show()
       
       #Create red images from threshold segmentation images
       new_out = np.zeros((512,512,3))
       for i in range(gray.shape[0]):
           for j in range(gray.shape[1]):
               if gray[i,j] != 0 and j>200:
                   new_out[i,j,0] = 255
       
       #Get green and red images overlapped
       print(k)          
       plt.figure(figsize = (10,10))
       plt.imshow(new_input)
       plt.imshow(new_out,alpha=0.5)
       #plt.savefig('E:\\Downloads\\DowloadsLatest\\UKY\\Research\\HorseBoneSegmentation\\Results\\ImageRegistration\\Hard To Be Good RF\\'+'Slice_'+str(k))
       
       #Get the evaluation metrics
       input_im = new_input[:,:, 1]
       output_im = new_out[:,:, 0]
       p00=0
       p01 =0
       p10=0
       p11 =0
   
       for i in range(input_im.shape[0]):
           for j in range(input_im.shape[1]):
               if input_im[i,j] == 0 and output_im[i,j] == 0:
                   p00 = p00 + 1
               elif input_im[i,j] == 0 and output_im[i,j] == 255:
                   p01 = p01 + 1
               elif input_im[i,j] == 255 and output_im[i,j] == 0:
                   p10 = p10 + 1
               elif input_im[i,j] == 255 and output_im[i,j] == 255:
                   p11 = p11 + 1
   
       print('p00 = '+str(p00))
       print('p01 = '+str(p01))
       print('p10 = '+str(p10))
       print('p11 = '+str(p11))
        
       if (p11+p10) != 0:
           pixel_accuracy = (p00 + p11)/(p00 + p01+p10+p11)
           mean_pixel_accuracy = 0.5 * (p00/(p00+p01) + p11/(p10+p11))
           IOU = p11/(p01+p10+p11)
           meanIOU = 0.5 *(p11/(p01+p10+p11) + p00/(p01+p10+p00))
           prec = p11/(p11+p01)
           recall = p11/(p11+p10)
           F1 = 2*p11/(2*p11 + p10 + p01)
           dice_coefficient = (2*p11)/(2*p11 + p10 + p01)
           metrics_list.append(['Slice'+str(k), pixel_accuracy, mean_pixel_accuracy, IOU, meanIOU, prec, recall, F1, dice_coefficient])
   
           print('pixel_accuracy = ' + str(round(pixel_accuracy,2)))
           print('mean_pixel_accuracy =' + str(round(mean_pixel_accuracy,2)))
           print('IntersectionOverUnion =' + str(round(IOU,2)))
           print('MeanIOU =' + str(round(meanIOU,2)))
           print('precision = ' + str(round(prec,2)))
           print('recall =' + str(round(recall,2)))
           print('F1 =' + str(round(F1,2)))
           print('dice_coefficient =' + str(round(dice_coefficient,2)))
   
           plt.text(250, 100, 'pixel_accuracy = ' + str(round(pixel_accuracy,2)), bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.text(250, 70, 'mean_pixel_accuracy =' + str(round(mean_pixel_accuracy,2)), bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.text(250, 40, 'IOU =' + str(round(IOU,2)), bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.text(250, 10, 'meanIOU =' + str(round(meanIOU,2)), bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.text(10, 10, 'precision = ' + str(round(prec,2)), bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.text(10, 40, 'recall =' + str(round(recall,2)), bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.text(10, 70, 'F1 =' + str(round(F1,2)), bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.text(10, 100, 'dice_coefficient =' + str(round(dice_coefficient,2)) , bbox=dict(fill=True, edgecolor='red', linewidth=2))
           plt.savefig('E:\\Downloads\\DowloadsLatest\\UKY\\Research\\HorseBoneSegmentation\\Results\\ImageRegistration\\Tizadaisy LF\\'+'Slice_'+str(k))
           plt.show()
       
   metrics_array = np.asarray(metrics_list)
   column_values = ['Slice','PA', 'MPA', 'IOU', 'mIOU', 'precision', 'recall', 'F1', 'dice_coefficient']
   df = pd.DataFrame(metrics_array, columns = column_values)
   filepath = 'E:\\Downloads\\DowloadsLatest\\UKY\\Research\\HorseBoneSegmentation\\Results\\MetricsExcel\\TizadaisyLF.xlsx'
   
   #df.to_excel(filepath, index=False)
   ```

   

   
