# Book-Cover-Via-Vocabulary-Tree
A project that uses a vocabulary tree approach to identify book covers from oblique images. It employs SIFT for feature extraction and a hierarchical vocabulary tree, built using hierarchical k-means clustering, for scalable and efficient recognition from a database of 100 book cover images. The approach reduces complexity and computation time by organizing visual features in a tree structure.

## Introduction

This project aims to identify the book covers of an image of book(s) taken in some oblique angle, from a database that contains around 100 book cover images scraped from Amazon. When given an image of some books, localising each individual book in the image and attempting to find the matching cover from the database seems to be the logical approach, but directly taking the features and descriptors of each book and comparing it against the entire database is very calculation heavy and complex, taking a lot of time and space.

Instead, the idea behind my approach for this is based on Nister and Stewenius' Scalable Recognition with a Vocabulary Tree and uses a vocabulary tree, which is a hierarchical structure for indexing visual features (descriptors of the image found by SIFT in this project). We will see details about how this tree is built, how the features are quantified and how the querying of an image of a book is done in this report.
 
Once the vocabulary tree is built, we can see how similar two different images are by querying the tree with their descriptors and finding the number of common clusters between them. It is explained in detail in the following sections. Although, before the images are compared, it is important to isolate/localise the books from the images.

After we have the top k matches to the queried image in the database, it can be filtered by applying RANSAC to find the best homography transformation between the queries image and each potential match given by the vocabulary tree. Finding the image comparison that has the highest total number of inliers, which are the number of consistent keypoints between the compared images, reduces the top k matches to just one, which should be the actual cover of the book.

## Localising Books from Query Images

Since the database for book covers I am using is already preprocessed and localised, only the query images which may contain more than one book have to be processed to somehow obtain each individual book in the image.

I first tried to use corner detection and edge detection, but neither of which gave satisfactory results and instead I ended up using the YOLOv8 (You Only Look Once) library with a built-in pretrained model to crop out different books from one singular image due to mainly the speed and accuracy with which it could recognise various objects since the model was trained on a general object detection dataset. 

After running the YOLOv8 model on the input image, we obtained bounding boxes around the detected books. These bounding boxes indicated the regions where books are located within the image. We can use these to crop out individual book regions from the original image, resulting in cropped images containing only one book each.
	
Take a look at the following examples of this below. Since I did not train this model on a custom dataset of book covers, it tends to make mistakes sometimes.

Original Image:\
![unnamed](https://github.com/user-attachments/assets/c3c795cd-e6ce-4302-8dc9-4492832546d3)

After running the YOLO model on the image:\
![image](https://github.com/user-attachments/assets/e7acfdcf-1abd-457a-b5de-896519913ab8)

Cropped image:\
![image](https://github.com/user-attachments/assets/d974b1c0-0c7c-4052-8c4a-ae83ec641f8c)

## Vocabulary Tree
 
The vocabulary tree is a hierarchical structure built using hierarchical k-means clustering on certain features (SIFT descriptors in this case). The process of building it begins by extracting local features,i.e., the SIFT descriptors from every image in a dataset of images. These represent distinct visual patterns in each image. Then, the descriptors are clustered hierarchically using k-means clustering, which organises the descriptors in a tree structure. Each level of the tree represents more detail than the last in the visual features.

Please read the report for detailed explaination about the math and theory behind this.\
![image](https://github.com/user-attachments/assets/3ae5f017-b1f9-4966-b1a2-c7ac8ee12344)

## Homography estimation with RANSAC

From the score computation, we can get the top 10 closest matches and perform feature mapping for each feature in the query image and find the closest feature in a potential final match image.

Once these are found, using RANSAC with 1000 iterations to find the best homography that has the highest number of inliers for each potential match to the query image and the highest inlier-holding image should be the book cover for that query image.

## Testing
![image](https://github.com/user-attachments/assets/ab866e55-520a-45bb-9dc9-99a2d668baf4)
![image](https://github.com/user-attachments/assets/50f011f7-471c-4abd-b47a-3ccdc364ca47)
![image](https://github.com/user-attachments/assets/21e84726-3605-4ed5-b69d-e32dd9cbc511)
![image](https://github.com/user-attachments/assets/85e54917-5a42-4dba-ae45-4cf9eae51a47)
![image](https://github.com/user-attachments/assets/390107f5-f905-47b9-bebc-3cc0231f342b)

## Challenges and Problems

The biggest issue was coming up with ideas to make the VocabularyTree setup and functions not be super heavy on memory and time complexity. The paper by Nister and Stewenius did not go in depth on how to implement the tree given space and time complexities and I had to run many tests to keep it as efficient as I could.
Another issue is that feature matching is a very slow task and can take up to a couple minutes to run in my implementation.

## Results

Overall, a lot of the predictions are correct. For the ones that aren’t, it seems to be a glare/reflection issue since I tried testing it with those exact same books under different lighting and similar angles, which can also be seen in some of the images above, that sometimes the book cover is accurately found and sometimes it is not.

Another problem is that yolov8 doesn’t always crop the image properly and sometimes there's another book or another object that gets counted as a book.

## References

[1] David Nister and Henrik Stewenius “Scalable Recognition with a Vocabulary Tree”, CVPR 2006\
[2] YOLOv8 https://github.com/ultralytics/ultralytics



