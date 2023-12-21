# NFL Player Contact Detection
This project was done as an assignment for the Machine Learning for Computer Vision course of the University of Bologna, we decided to pick a Kaggle [competition](https://www.kaggle.com/competitions/birdclef-2023/overview) and work on it.
The aim is to detect contacts between players during NFL games. We utilize both tracking data and video feeds to achieve this. To simplify the problem we've excluded contacts involving players not visible on camera and contacts with the ground.
Below you can find a summary of the project, which includes the main objectives, methods and results. For more details, you can check out the notebooks that contain the code and analysis.

## Data

The following data sources were avaiable:

- Two video feeds from different angles: Sideline and Endzone, recorded at a frame rate of 59.94Hz
- Bounding boxes representing the location of player's helmets
- Player tracking data, recorded at a frequency of 10Hz

## Pre-Processing

We've pre-processed the data to ensure its relevance and accuracy. This involved:

- Filtering out irrelevant data
- Removing inconsistent data entries
- Excluding contacts with the ground
- Merging labels, distances between players, and helmet bounding box data into a unified dataframe
- Computing bounding boxes for contacts

## Contact Boxes
In order to detect the presence of a contact between two players we will have to focus on the portion of the image interested by the contact. We can find it by using the information of the bounding boxes and making the following assumptions:

- The contacts will likely happen in the space between the two players, i.e. in the space bewteen the helmets
- In football the contact often happens between hands vs shoulders or hand vs chest. For this reason we need to consider also the region below the helmets
  
Therefore we calculate the region between the helmets by merging the boxes of the helmets.

![Contact Boxes](./images/Contact_Boxes.png)

## Masking

We've applied masking to exclude all contacts where:

- One or both players are not visible
- The distance between the two players exceeds the 99th percentile of the distances observed in actual contacts

## Memory Management

To ensure efficient memory usage, we've implemented the following measures:

- Reduced the resolution of the video from 720x1280 to 360x640
- Reduced the number of frames analyzed by sampling one frame every three
- Converted and stored data in the Hierarchical Data Format (HDF)

## Networks

### ForkNet SIML (Single Image Multi Label)

ForkNet SIML is a network designed to classify multiple regions within each input image. Each image contains several regions, defined by potentially overlapping bounding boxes.

While this task is not strictly object detection, it shares several characteristics with it:

- The goal is not to classify the entire image, but specific portions of it.
- Each image contains multiple regions (approximately 10 on average) that need to be classified.
- The regions of interest are defined by bounding boxes, which can overlap.

However, there are key differences compared to a traditional object detection problem. For instance, we already have the contact boxes, eliminating the need for a Region Proposal Network. Additionally, our task is not to classify objects, but to detect whether a contact has occurred.

Given these considerations, we chose to use a network inspired by Fast R-CNN with a ResNet50 backbone due to its proven effectiveness in similar tasks.
Each frame is processed by a feature extractor made of the first 3 freezed layers of ResNet50. The feature extractor produces a feature map that, together with the contact boxes, is processed by our OneViewNetwork. This net takes the contact boxes and maps them onto the feature map (roi-pooling). The output of this operation is then processed by the PerRegionNetwork which decides whether a contact took place or not.

As explained before, to mantain dimensional consistency in every batch, each example consists of all the 231 combinations of players. We use the mask to perform roi-pooling and detection only to the non-masked regions (related to visible and close players). Therefore, the output of the OneViewNetwork will be a vector of lenght 231 having zero value for masked regions and the logits outputted by the PerRegionNetwork for all the other players. Note that also in the computation of the loss the output of the masked regions is ignored.

Another complication intrinsic in this task is that we have the information related to two cameras, Sideline and Endzone. We decided to work at increasing levels of complexity.

Firstly, we processed the inputs using just one of the views to set up a network baseline and to compare the informative power of the two views.
Secondly, we tried to merge the outputs of the OneViewNetwork with a linear layer. In this second version the networks (OneView- and PerRegion-Network) of both views shared the same parameters.
Finally, we instantiated different networks for each view.

The input/ouutput structure of the network:

![SIML Input](./images/Siml_input.png)

Below an overview of the network architecture:

![SIML Input](./images/Siml_architechture.png)

### ForkNet SISL (Single Image Single Label)

This network takes the following as input:

- A 128x128 cropped input image (Sideline and Endzone)
- A 128x128 binary image of the contact box (Sideline and Endzone)

![SISL Input](./images/SISL_input.png)

The following image illustrates how the mask works:

![SISL Maks](./images/Sisl_mask.png)

The image below provides an overview of the network architecture:

![SISL Architechture](./images/Sisl_architechture.png)

## Results

The table below presents the results of our analysis:

![Table of Results](./images/Table_of_results.png)
