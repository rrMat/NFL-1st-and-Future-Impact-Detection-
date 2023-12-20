# NFL Player Contact Detection

This project aims to detect contacts between players during NFL games using both tracking data and a video feed. To simplify the problem, we have ignored contacts of players not on camera and contacts of players with the ground.

## Data

The project uses the following data:

- Two video feeds from different angles: Sideline and Endzone, recorded at 59.94Hz
- Bounding boxes of player's helmets
- Player tracking data recorded at 10Hz

## Pre-Processing

The data was pre-processed by:

- Filtering out irrelevant data
- Removing inconsistent data
- Excluding contacts with the ground
- Merging labels, distance between players, and helmet bounding boxes into a single dataframe
- Computing contact bounding boxes

## Contact Boxes

*Add contact boxes image here*

## Masking

We have masked all contacts where:

- One or both players are not visible
- The distance between the two players is larger than the 99th percentile of the distances of actual contacts

## Memory Management

To manage memory efficiently, we have:

- Reduced video size from 720x1280 to 360x640
- Reduced the number of frames analyzed by sampling one frame every three
- Converted and stored them as Hierarchical Data Format (HDF)

## Networks

### ForkNet SIML I

This network classifies portions of the input images. For each image, there are many regions to be classified. These regions are defined by overlapping bounding boxes.

The input is as follows:

*Insert image here*

The following image depicts the network as a whole:

*Insert image here*

### ForkNet SISL

The input for this network is:

- 128x128 cropped input image (Sideline and Endzone)
- 128x128 contact box binary image (Sideline and Endzone)

*Insert images here*

The following image depicts the network as a whole:

*Insert image here*

## Results

The following table presents the results:

*Insert image here*
