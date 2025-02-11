### Youtube Teller -  a  Video StoryTelling model to extract people's daily activity

### Description
Video Description/Storytelling is a complicated task which involves automatically captioning a video by understanding the action and event in the video which can help with the retrieval of the video efficiently through text task. For this final project, we are trying to solve a video-caption problem which specifically focuses on video captioning tasks on Youtube video. <br>
Our original plan was to focus on cooking activity - caption paring dataset and generate a cooking guide from that dataset. But after we realize that that dataset is really hard to be preprocessed as well as with the previous work related to it is very little, we switch to this [YoutubeClips dataset](https://www.cs.utexas.edu/users/ml/clamp/videoDescription/).

### DataSet
[YouTubeClips dataset](https://www.cs.utexas.edu/users/ml/clamp/videoDescription/)

Download both the `Microsoft Research Video Description Corpus` and `YouTubeClips.tar` from the above link.

#### Form
The Video is named using its id: For example: `-4wsuPCjDBc_5_15` where `_5_15` means from the 5th second to the 10th second.

![](https://github.com/MRSA-J/Youtube-Teller/blob/main/readme%20image/video%20form.png)

The Description txt is in the following form: <br>
```
-4wsuPCjDBc_5_15 a squirrel is eating a peanut in it's shell 
-4wsuPCjDBc_5_15 a chipmunk is eating 
-4wsuPCjDBc_5_15 a chipmunk is eating a peanut 
-4wsuPCjDBc_5_15 a chipmunk is eating a nut 
-4wsuPCjDBc_5_15 a squirrel is eating a nut 
-4wsuPCjDBc_5_15 a squirrel is eating a whole peanut 
-4wsuPCjDBc_5_15 a squirrel is eating a peanut 
...
```

With the `-4wsuPCjDBc_5_15` being the video id and `a squirrel is eating a peanut in its shell` being the description. Since the captions all have similar meanings, this task is more like a video caption task instead of a storytelling one.


#### Usage 
1. After downloading the dataset, create a folder `data` which is in the same hierarchy with our code and put the unzipped version of `YouTubeClips.tar` and `Microsoft Research Video Description Corpus` inside the data folder.
2. run `python convert_video_to_image.py`
This helps convert our video to sequence of images
3. run `python preprocess.py` to preprocessing
This helps us build our training/testing/validation dataset
4. Set the parameters and train the model like the following
`python train.py --only_prefix --out_dir ./msv_train/ --mapping_type transformer  --num_layers 8 --prefix_length 40 --prefix_length_clip 40`
5. Use the trained model to predict result
`python predict.py`

### Methodology
We choose to treat this task as a sequence-to-sequence generation task.<br> 

To start, we used the `cv2` package to help us turn video into a sequence of image frames. We set the frame rate to be 1s per image frame, as the videos in our dataset are generally 5 or 10 seconds in length. After several tests, the frame rate that 1s per image frame is adequate for this translation task, and also economical since it has the lower number of images generated to be fed into our model. Considering the large size of YoutuberClips dataset, we limited the number of images that can be extracted from each video. In this process, we keep a good balance of the trade off between the number of images we extracted and the quality of the model performance.Feel free to make a larger image frame set corresponding to the video if you have more time. <br>

![](https://github.com/MRSA-J/Youtube-Teller/blob/main/readme%20image/video-image.png)

In the next step, we used 3 different strategies to process our image frames, so that it can be put into the model. Below are the 3 ways of input (and we will justify our choice in our report):

1. Turn our video into `random single image` frames.
2. Turn our video into `mean image` frames. That means the mean of every image frame corresponding to that video.
3. Turn our video into `sequential image` frames. And we will use positional encoding afterwards in our transformer part to deal with it.

The model is inspired by the idea of [ClipCap: CLIP Prefix for Image Captioning](https://arxiv.org/pdf/2111.09734.pdf). Their model structure is like the below. We removed the pretrained COCO dataset’s weight and applied our own creativity (positional encoding) on the model.We make this choice, as we believe using prefix training to capture the video/image feature is a very lighted way of training and can achieve good results without the need of managing to "train too much". And instead of passing the image, we pass the preprocess video (image frames) into the CLIP model.

![](https://github.com/MRSA-J/Youtube-Teller/blob/main/readme%20image/ClipCap%20Model.png)

In our model we applied the Transformer-encoder Mapper model structure to encode image information into embedding space, so that the GPT-2 is suitable for this special task without training. The CLIP model is designed to impose a shared representation for both images and texts. After training over a vast number of images and textual descriptions using the cross entropy loss, the visual feature of CLIP and textual representations of GPT-2 are well correlated. Therefore, the fine-tuned Mapper structure allows prefix embeddings to capture the visual information, and effectively generate the suitable input for the GPT-2 model to correctly predict. <br>

Our modified model structure looks like:

![](https://raw.githubusercontent.com/MRSA-J/Youtube-Teller/main/readme%20image/Model%20Structure.png)

The whole process of this model is shown above. We preprocess videos into image embeddings. And then we concatenate image embedding with prefix embeddings. Then we feed such combinations into the Feature Mapper model to allow prefix embeddings gaining enough visual information. Then, we only maintain the prefix embedding parts and feed them into the GTP-2 model to generate the final predictions.

### Evaluation Metrics
We plan to test our video caption model on the test dataset of uncaptioned videos to generate their captions. We evaluate the performance of our model on the similarity of the generated sentences and standard answers. Specifically, we adopt 4 metrics to evaluate the accuracy of our captions, namely, BLEU, CIDEr, METEOR, and Rouge. 

The baseline model (Vision Transformer) can achieve 68.4 1-gram BLEU score and 50.7 5-gram BLEU score. We hope to improve the performance in some specific subjects, to achieve higher BLEU scores than the baseline model.

### Our Generation Examples
| Video                         | 1                 |2             | 3               | 4               |
| ----------------------   | ----------- |----------- |----------- |----------- |
| Video ID          |  `0bSz70pYAP0_5_15`  | `-vg3vR86fu0_1_6` | `60x_yxy7Sfw_1_7`| `9HDUADeA2xg_3_31` |
| Sample Image Frame |![](https://raw.githubusercontent.com/MRSA-J/Youtube-Teller/main/readme%20image/sample%20video%20image/0bSz70pYAP0_5_15/0bSz70pYAP0_5_15image10.jpg)|![](https://raw.githubusercontent.com/MRSA-J/Youtube-Teller/main/readme%20image/sample%20video%20image/-vg3vR86fu0_1_6/-vg3vR86fu0_1_6image4.jpg) | ![](https://raw.githubusercontent.com/MRSA-J/Youtube-Teller/main/readme%20image/sample%20video%20image/60x_yxy7Sfw_1_7/60x_yxy7Sfw_1_7image5.jpg)|![](https://raw.githubusercontent.com/MRSA-J/Youtube-Teller/main/readme%20image/sample%20video%20image/9HDUADeA2xg_3_31/9HDUADeA2xg_3_31image13.jpg)|  
| All Image Frame |[Plane video image frame](https://github.com/MRSA-J/Youtube-Teller/tree/main/readme%20image/sample%20video%20image/0bSz70pYAP0_5_15) |[Man riding image frame](https://github.com/MRSA-J/Youtube-Teller/tree/main/readme%20image/sample%20video%20image/-vg3vR86fu0_1_6)|[Man watch video of woman image frame](https://github.com/MRSA-J/Youtube-Teller/tree/main/readme%20image/sample%20video%20image/60x_yxy7Sfw_1_7)|[Doggie playing image frame](https://github.com/MRSA-J/Youtube-Teller/tree/main/readme%20image/sample%20video%20image/9HDUADeA2xg_3_31)|
| Sample Ground Truth Sentence|an airplane is flying in a wide circular pattern.|a guy on a bicycle who tries to jump his bike on a wooden ramp.|a man seated is watching and admiring the image of a woman on the screen of his laptop.| a puppy is playing with a ball.|   
| All Ground Truth Sentences|[Plane video captions](https://github.com/MRSA-J/Youtube-Teller/blob/main/readme%20image/sample%20video%20image/0bSz70pYAP0_5_15.txt) |[Man riding captions](https://github.com/MRSA-J/Youtube-Teller/blob/main/readme%20image/sample%20video%20image/-vg3vR86fu0_1_6.txt)|[Man watch video of woman captions](https://github.com/MRSA-J/Youtube-Teller/blob/main/readme%20image/sample%20video%20image/60x_yxy7Sfw_1_7.txt) | [Doggie playing captions](https://github.com/MRSA-J/Youtube-Teller/blob/main/readme%20image/sample%20video%20image/9HDUADeA2xg_3_31.txt) |   
| Single                           | a flying airplane flying over a lake.| a man is riding a motorcycle. | a woman is dancing.| a baby is walking on a rug. |   
| Mean                            |  a plane is flying.  | a man is driving a car.   | a man is talking on a computer. | a cat is playing with a toy.    | 
| Sequential                   |  a plane is flying. |a man is riding a motorcycle.|a man is talking to a woman.| a cat is playing with a toy.| 

As above, single, mean, sequential means different preprocessing methods and how image frames are selected and passed to the model. 

| Method       |      BLEU      |   CIDEr             | METEOR               | ROUGE               |
| ----------------------   | ----------- |----------- |----------- |----------- |
| Single     |  0.487  | 0.465 |  0.281 |  0.630 |
| Sequence   |  0.507  | 0.549 |  0.297 |  0.641 |
| Mean       |  **0.520**  | **0.584** |  **0.302** |  **0.647** |


We will analyze this result in our report.

### Contributor
Chen Wei (cwei24), Yuan Zang (yzang6), Yunhao Luo (yluo73)

### Division of labor 
We plan on working equally across X aspects of the project:
1. Preprocess the data: Chen Wei, Yuan Zang
2. Model Architecture
  - Caption Generater (Use GPT2 pretrained model): Chen Wei, Yuan Zang
  - Video/image CLIP encoder: Yuan Zang
  - Multi-head attention Transformer: Yuan Zang, Chen Wei
  - Transformer with Positional Encoding to encode position information: Yuan Zang
4. Evaluation (BLEU, METEOR, CIDEr, ROUGE) and and Visualization: Yunhao Luo
5. Model Training: Yuan Zang
6. Ablation study: Chen Wei
7. Write the report and make the poster: Chen Wei

### Ethics
##### What broader societal issues are relevant to your chosen problem space?
In this project, we aim at generating high-quality, articulated text descriptions given videos as input.  To this end, we can improve the accessibility of various videos in the wild, which hopefully can benefit  users. By the generated descriptions/tags, we can also sort and categorize massive video sets.
##### Why is Deep Learning a good approach to this problem?
Deep learning is currently the most popular and accurate method for computer vision/natural language processing. As for video understanding, by using neural networks with convolutional/attention, the model can learn effective representation. In addition, deep learning methods can achieve end-to-end modeling and are more flexible than traditional methods that usually use handcrafted features (lacking generalizability to  other datasets). 

### Related Work
- [ClipCap: CLIP Prefix for Image Captioning](https://arxiv.org/pdf/2111.09734.pdf)

### Reflections
Our project ultimately turned out to be ok and our model works as expected. It can generate captions that are acceptable and coherent although not being perfect. <br>
If we have more time, we could implement grid search on the number of image frames we will use to represent a video and the frame_rate parameters when extracting image frames from videos to make the result better. Also, we could try different model structures and do more ablation studies. For example, using C3D to extract video structure might perform better than our current model, as it preserves positional information better.
