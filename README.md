# DNN-Aided-Step-Detection

**Supervised by: Dr. Nir Shlezinger**


**1** **Introduction**

 In robotics, gait planning relies heavily on the knowledge whether
 each foot touches the ground. For that purpose, sensors are placed on
 each leg tip, indicating whether or not a step occurred. Being able to
 detect a step by using the IMU measurements, might allow redundancy of
 the force sensor (or any step detection sensor) in robotics --
 replacing hardware with a software algorithm.

 When a pedestrian walks, its foot accelerates in a repetitive,
 periodic pattern. This project assumes that each part of the period
 indicates a different phase of the walk, i.e., lift foot, swing foot,
 step etc. Using recorded and labelled data from a pedestrian walk,
 this project aims to predict whether or not a step occurs - using IMU
 (Inertial Measurement Unit) measurements only.

 By looking at the future contribution of this project (in another
 domain), being able to detect and count steps could lead to the
 ability of tracking a user's healthcare of any person having a
 smartwatch or any other IMU including device.

 **2** **Dataset**

 This chapter describes the technical information of the dataset used
 in this project and its preprocessing.

 **2.1** **Dataset collecting**

 The quality of the dataset influences the ability to receive good and
 reliable results, and thus a major part of the work was invested in
 achieving a proper dataset. The team first tried to generate a dataset
 from a 4-legged robot simulation -- which failed due to reasons such
 as technical issues such as labeling the data, working with an
 unfamiliar simulation etc. Finally, a pedestrian walking dataset
 including a reliable labelling was found and would further be
 described.

 **2.2** **Data description**

 This section discusses the general information about the used dataset.

 **2.2.1** **IMU Relevant background**

 IMU is a sensor used to measure three perpendicular linear
 accelerations and another three perpendicular angular accelerations.
 The IMU is widely used both in rare devices and in everyday used
 devices (such as cell phones, smartwatches, autonomous vehicles etc.).
 The IMU used to collect the used dataset in this project -- the
 Pedometer Dataset, is called "Shimmer3" and can be seen below, along
 with its axis of measurements:
 
![](media/image1.png)

**Figure 2.1: Shimmer3 IMU**

 This IMU is a MEMS (Micro Electrical Mechanical System) based on a
 "mass-spring" working principle, as can be seen below:
 
![](media/image2.png)

**Figure 2.2: mass-spring system**

 Whereas in order to use such a system to measure acceleration one may
 use Newton's law:

𝐹 = −𝑘𝑥 = 𝑚𝑎 = 𝑚𝑥̈

 Leading to the measurement equation:
 
  𝑎 = − 𝑘 / 𝑚 * 𝑥

 Denote, that this is only the principle, and todays IMU's are more
 advanced and complicated.

 **2.2.2** **Pedometer Dataset collection**

 The dataset used in this project is based on Pedometer Dataset \[1\]
 collected by Ryan Mattfeld. In order to fully understand the data and
 the way it was collected the group met Prof. Mattfeld online.

 This project focused on the regular walk, as we are interested only in
 step detection although the dataset includes IMU recordings of
 regular, semi-regular and irregular walks. The IMU sensors were placed
 on the participant's wrist, hip, and ankle -- each including 6 axis
 sensors (3 linear accelerometers and 3 gyroscopes), as can be seen
 below:

![](media/image3.png)

**Figure 2.3: IMU\'s locations**

 The axes were positioned in the following manner:

![](media/image4.png){width="1.8097211286089239in"
height="2.5833333333333335in"}

**Figure 2.4: axes positions and orientations**

 **2.2.3** **Data Labelling**

 In order to label the data, a synchronized video was captured along
 with the IMU\'s recordings, as demonstrated in the following figure:

![](media/image5.png)

**Figure 2.5: Video capturing demonstration**

 As a part of the work done in order to label the data, the ground
 truth step labeling was done \"by hand\" using a developed tool for
 visualizing the IMU data next to the recorded video. It is important
 to note that a step was labeled at the exact time when the front part
 of the leg touched the ground at a discrete moment.

![](media/image6.png)

**Figure 2.6: labeled steps data**

 Figure 2.6 shows the labeled steps, with red and blue lines indicated
 a right and left leg respectively. An example for the data is given in
 the python notebook added to this project.

 Some of the labelled data could be seen in the following figure:

 **2.2.4** **Data plot**

The first five indices of the plot of a single person, from a single
sensor is shown below:

![](media/image7.png)

**Figure 2.7: Data head(5)**

The labels list was given as a text list, indicating the index of the
step and which leg

 stepped (right or left) as can be seen:

![](media/image8.png)

**Figure 2.8: labels list example**

 The IMU sensor recordings and labels were plotted vs. their time
 index:

![](media/image9.png)

![](media/image10.png)

**Figure 2.9: IMU sensor recordings and labels**

 **2.3** **Data processing**

 The data processing is a crucial stage before even beginning to plan
 the NN architecture and hyperparameters. This stage allows the
 designer to get a good intuition on the way the data behaves.
 Furthermore, the preprocessing allows "helping" the neural network to
 be able to learn the data, as the designer should simplify the data
 before learning process.

 **2.3.1** **Synchronization**

 The collected data was labelled by index corresponding to the last
 sensor turned on. This sensor was found by taking the latest time in
 the first value of the "date" column. In order to synchronize the
 labelled indexes to the ankle sensor, the recordings collected before
 the last sensor was turned on were erased. This was also verified with
 Prof. Mattfeld, who collected the data, in order to make sure the data
 synchronization was done perfectly -- as it is highly important and
 would affect the results directly.

 **2.3.2** **Window size selection**

 In order to slice the time series data into sub-series, an appropriate
 time window size was examined by slicing the data into sub series each
 one between two steps. The amount of each window size in the dataset
 is shown below:

![](media/image11.png)

**Figure 2.10: window size histogram**

 From the graph above it can be seen that between two labeled steps
 there are usually around 8 samples. Therefore, the data was divided
 into two classes, one for a step (left leg) and one for a non-step.

![](media/image12.png)
**Figure 2.11: sub-series slicing**

 The window is placed in such a way that there are n samples before the
 labeled step and m samples after the step (when n and m are
 hyperparameters - at the figure below n=0, m=4).

 **2.3.3** **Data whitening transformation**

 The IMU measurements were whitened as their absolute value means less
 than their values relative to each other -- helping the NN train
 better and be able to generalize. A whitening transformation is a
 linear transformation which standardizes features by removing the mean
 and scaling to unit variance. It was done since some features that are measured at different scales do not
 contribute equally to the learning process and might end up creating a
 bias.

 **2.3.4** **Single person vs. multiple people**

 Thanks to the fact that the data was collected per person and divided
 into different sub-datasets (each for every person) including 30
 people, two different approaches could be examined -- the first for a
 step detection based on a dataset of a single person and the other on
 the dataset of multiple people. The multiple people dataset was mixed
 without distinguishing between people between the training,
 validation, and test in order to allow model generalization.
 Furthermore, each multiple people dataset consisted of five random
 persons (as it was our upper bound due to calculating and memory
 issues).

 **2.3.5** **Data distribution examination**

 In order to get a feeling of our collected data the whitened data
 distribution of one person was plotted for each class and for each
 sensor, as can be seen below:

![](media/image13.png)

**Figure 2.12: Data distributions -- single person**

 The figure above allows analyzing the dataset's characteristics by
 illustrating the statistics of different aspects of the dataset. The
 same distribution examination was done for a data that was collected
 from multiple people's sensors:

![](media/image14.png)

**Figure 2.13: data distribution - multiple people**

 To distinguishing between the different people, the following figure
 shows the distributions of each sensors of some different persons:

![](media/image15.png)
**Figure 2.14: measurement distributions of different persons**

 It can be seen from the graphs that different people have slightly
 different distributions which might make it difficult for the network
 to learn to recognize steps in a general way.

 By physically analyzing a regular walk, one may expect that the
 angular acceleration be more dominant around the z axis, which is the
 ankle's axis of rotation (as defined in Figure 2.4). By understanding
 this physical expectation, it can be inferred that gyroscopes x and y
 are expected to result in noisy measurements with zero-mean (no
 tendency). An example for this noisy behavior can be seen in the
 following figure:

![](media/image16.png)

**Figure 2.15 Noisy behavior of gyroscope y**

 This behavior could also be seen in Figure 2.12 where the
 distributions of gyroscopes x and y are approximately normally
 distributed around zero.

 **3** **Limitations**

 This chapter describes the main limitations of the project.

 **3.1** **Sensor limitations**

 This section focuses on the limitations related to the sensors that
 were used to collect the data.

 **3.1.1** **Saturation**

When observing the accelerometer measurement, a high saturated value
could be seen:

![](media/image17.png)

**Figure 3.1: IMU Accelerometer y Saturation**

 This saturation is also expressed as an exceptionally high bean could
 be seen at the linear accelerometer along the X and Y axes. These
 beans could possibly be associated with a sensor saturation value. In
 order to confirm it is really a saturation phenomenon, the Shimmer3
 IMU's datasheet was examined, as shown in appendix 9.1. This could
 explain the high count at that exact value. As shown in Figure 2.4 the
 Y axis is along the "up-down" direction along gravity, and the X axis
 is the footstep direction.

 **3.1.2** **Non-rigid sensor position and orientation**

 The sensors are placed on the person's ankle by using a strap, as
 shown in Figure 2.3. This leads to variance between different people
 that may have placed the strap and the sensors in slightly different
 ways. The use of an elastic strap to attach the sensor to the ankle
 may lead to a degree of freedom and relative motion between the sensor
 and the ankle.


 **3.1.3** **Measurement noise**

 As explained, the dataset is based on sensors' recordings -- which
 have internal measurement noises from many different reasons such as
 temperature, warm-up, etc. which directly affect the measured values.
 These noises lead to an uncertainty in figuring out the true
 acceleration and give rise to many challenges when using the sensed
 data. The IMU's datasheet is shown in appendix 9.1.

 **3.1.4** **Low sampling frequency**

The sensors' recorded frequency is 15\[𝐻𝑧\] which is a low frequency
compared to the

system's dynamics, as there is a step at 2\[𝐻𝑧\] (around a step per leg
per second). This

 gives rise to a major challenge of slicing the data into more than 2
 classes, for example dividing the data into left step, right step and
 no step - as there will be overlapping windows that are labeled
 differently.

 **3.2** **Labelling**

 This section discusses the labelling limitations.

 **3.2.1** **Manual discrete labelling**

 The steps were detected by a pre-developed tool that allowed a person
 viewing a recorded video, to tag a specific moment a step occurred
 (right or left) -- from the recorded data. Labelling by hand in such a
 way might lead to a variety of errors: timing errors, accuracy errors,
 frequency limitations and more. Moreover, a step is not a
 deterministic value, and thus two different people could tag a step at
 two slightly different times.

 **3.2.2** **Labelling visually**

 The labelling tool described above forced the person labeling to
 visually inspect a recorded video and conclude whether or not a step
 occurred at every recorded moment.

 This method is highly sensitive to errors, for example:

 1.The video was shot from a specific angle which might have limited
 his/her point of view.

 2.The visual labeling is limited to the camera's resolution and has no
 strict conditions that determine a step.


 **4** **Network**

 As the dataset is a timeseries, we chose to compare two leading
 options:

 1.RNNs -- GRU: these types of networks hold a hidden layer that allows
 the network to treat the data as a time series and "weight" the
 dependency of timestep on another.

 2.CNNs: A network that is usually used to analyze images but is also
 used in the field of time series analysis -- by spanning the time
 measurement along a specific dimension, the different sensors' axes
 along another dimension and the different sensor types along the
 channels. This construction allows real time implementation due to the
 relatively short runtime of the CNNs.

 **4.1** **CNN1D**

 This chapter discusses the first proposed architecture -- CNN1D.

 **4.1.1** **General**

 One dimensional convolutional neural network is very similar to a
 fully connected layer, but with the difference that each output is the
 weighted sum of a few (size) input values thar are close (dilation) to
 each other. The innovation in this method is that the weights are kept
 constant when "sliding" along each input and are being updated along
 the backpropagation. An illustration for the different parameters of
 the 1d CNN is shown below:

![](media/image18.png)

**Figure 4.1: illustration of 1D CNN \[2\]**

 This figure illustrates the stride, kernel size, and dilation.


 **4.1.2** **Method**

 In order to allow the network to "sense" the step through the change
 in the sensors' measurement along time, the input data was organized
 in the following manner:

![](media/image19.png)

**Figure 4.2: CNN1D convolution direction**

 This way, the convolution is able to extract features from the time
 dimension, by using all sensors data in a given batch.

 **4.1.3** **Hyperparameters**

 The following hyperparameters were used:

 ●Data window characteristics:\
 oType -- constant side, full stride (not a sliding window). oSize -- 5
 timesteps.

 ●Training hyperparameters:\
 oBatch size:32.

 oOptimizer: Adam.\
 oLearning rate: 8e-2.

 oScheduler: Exponential, 𝛾=0.9.

 oLoss function: Cross Entropy.

 ●Network inherent hyperparameter:\
 oNumber of CNN1D layers: 1.\
 oNumber of FC layers: 3.

 oNumber of Flatten layers: 1.\
 oActivations: relu, 2.

 oKernel Size: 3.\
 oStride:1.

 oPadding: 0.\
 oDilation: 1.

 **4.1.4** **Architecture**

 The following figure illustrates the architecture used:

 ![](media/image20.png)

**Figure 4.3: illustration of the CNN1D NN architecture**

 Note, that the horizonal axis represents the batch axis, and thus
 includes 32 more data columns, indicated by the "triple dots".


 **4.2** **RNN -- GRU**

 This chapter discusses the second proposed architecture -- GRU.

 **4.2.1** **General**

 Gated recurrent units (GRU) are an improved version of standard
 recurrent neural networks and they aim to solve the vanishing and
 exploding gradient problem which is a major shortcoming of standard
 recurrent neural networks. An illustration of the GRU is shown below:

![](media/image21.png)

**Figure 4.4: GRU illustration \[3\]**

 To solve the problem described above, GRUs use update gate and reset
 gate. Basically, these are two vectors which decide what information
 should be passed to the output. The special thing about them is that
 they can be trained to keep information from far behind (in time),
 while removing information which is irrelevant to the task.

 **4.2.2** **Method**

 GRUs store and filter the information using its gating mechanism. It
 can be useful to insert a segment of a time sequence and analyze its
 encapsulated information by enlarging its output dimension (i.e.
 features), enabling us to spot some non-trivial correlations among the
 given features. The output of the GRU layer is a stationary vector of
 the given time sequence input (we take the last hidden state as an
 output), so we can consider it as a representative summation of the
 input, then we feed forward a quite simple fully connected layers,
 ending up with a reliable candidate of the input true classification.

 **4.2.3** **Hyperparameters**

 The following hyperparameters were used:

 ●Data window characteristics:\
 oFeatures: 6.

 oSize: 5 timesteps.

 ●Training hyperparameters:\
 oBatch size:64.

 oOptimizer: Adam.\
 oLearning rate: 8e-2.\
 oweight_decay:1e-3

 oScheduler: Exponential, 𝛾=0.8.\
 oLoss function: Cross Entropy.

 ●Network inherent hyperparameter:\
 oNumber of GRU layers: 1.\
 oNumber of FC layers: 3.\
 oFC activation: tanh.

 **4.2.4Architecture**

 The following figure illustrates the architecture used:

![](media/image22.png)
**Figure 4.5: GRU architecture**

 The GRU feature size input is 6 and its output hidden state is of 128
 features (we take the last hidden state as an input to the FC layers),
 the first FC layer receives 128 features and its output is 64
 features, activated by hyperbolic tangent function.

 The second FC layer receives 64 features, and its output is 16
 features, activated by hyperbolic tangent function, the last FC layer
 output is two-dimension inactivated vector.

 **5** **Benchmark**

 This work is evaluated compared to Basil Lin's thesis \[4\] which also
 used the Pedometer dataset \[1\] together with a CNN to predict the
 step counts in a selected window. Lin's work aims to solve a different
 problem than this project, but we found it to be the best benchmark to
 evaluate our work.

 The author used a CNN with a sliding window of data in order to
 predict the number of steps in each window -- allowing step counting
 by using the trained NN to count steps in a flow of new IMU data. The
 data was shaped in such a way that each sensor is represented as a
 channel, and the width of the image is occupied by the timeseries in
 the length of the sliding window. The work examined a few gait types,
 considered many different window sizes and 3 different sensor
 locations: ankle, hip, and wrist.

 The resulted accuracy of evaluating the number of steps in a given
 section was concluded with 98% (30,897 steps were counted out of
 31,528 actual steps) in the regular walk. It is important to note that
 these results are different than our expected results -- as our
 mission is to detect whether a step occurred, rather than counting the
 total number of steps conducted.

 **6** **Results**

 This chapter will present the results obtained in the work.

 **6.1** **One Person Examination**

 This section describes the results received by taking the data only
 from a single person walk recording -- for all train validation and
 test:

 **6.1.1CNN**

 The 1-dimensional convolution network handled the challenge of
 learning step detection. This could be seen in the figure below:

![](media/image23.png)

**Figure 6.1: CNN1D single person - learning curve**

 As can be seen, the cross-entropy loss function decreases with the
 advancing epochs -- implying that the network "learns" to output a
 value closer to the labelled true value expected. The main trend is a
 sharp decrease in the loss, indicating the quick learning of the
 network. As can be seen, there is a small "peak" in the curve, which
 could be due to the stochasticness of the gradient descent. The final
 network accuracy when tested on the "test dataset" was **100%**. This
 indicates that the network detected correctly 100% of the step
 occurred in the test data.

 **6.1.2** **GRU**

 The Gated Recurrent Network (GRU) also handled the challenge of
 learning step detection. This could be seen in the figure below:

![](media/image24.png)

**Figure 6.2: GRU single person - learning curve**

 In the same manner as resulted in the CNN1D, the cross-entropy loss
 function decreases with the advancing epochs. The main trend is a
 sharp decrease in the loss (but not as sharp as the CNN1D curve),
 indicating the quick learning of the network. As can be seen, there
 are some "peaks" in the curve, which could also be due to the
 stochasticness of the gradient descent. The final network accuracy
 when tested on the "test dataset" was **100%**. This indicates that
 also this network architecture detected correctly 100% of the step
 occurred in the test data.

 **6.1.3t-SNE test set representation**

 t-SNE (t-distributed stochastic neighbor embedding) is a popular
 dimensionality reduction technique. Since our features across the
 network layers are characterized

by 𝑛 features, 𝑛 ≥ 2, we have to reduce the feature dimensionality in
order to display

 them. t-SNE generates a lower number of features (typically two, as
 here) that preserve the relationship between samples.

 The following figure displays the separation of the step and not-step
 classes, for one person:

![](media/image25.png)
**Figure 6.3 t-SNE representation of a single person dataset - CNN1D
(left) GRU (right)**

 It can be seen that there is a full separation, implying that 100%
 accuracy is expected to be achievable (and was also achieved).

 **6.2** **Multiple People Examination**

 This section describes the results received by taking generalized
 data, in the sense of using multiple people walks recording -- for all
 train validation and test. Few experiments were conducted, such that
 in each experiment eight random people were chosen as a representative
 group to evaluate the network. This chapter shows a specific
 experiment's results and a comparison between a few experiments is
 demonstrated at the end of this chapter.

 **6.2.1CNN**

 The 1-dimensional convolution network handled the challenge of
 learning generalized step detection. This could be seen in the figure
 below:

![](media/image26.png)

**Figure 6.4: CNN1D multiple people - learning curve**

 As can be seen, the cross-entropy loss function of the trained data
 decreases with the advancing epochs -- implying that the network keeps
 "learning" to output a value closer to the labelled true value
 expected -- in the training data. In contrast to that, the validation
 saturates after around \~10 epochs, implying that the network stopped
 generalizing at around that stage. The step detection accuracy
 received when comparing the output to the label on the multiple people
 dataset at this experiment was **94.1%**.

 **6.2.2** **GRU**

 The GRU network also handled the challenge of learning generalized
 step detection. This could be seen in the figure below:

![](media/image27.png){width="2.6125in"
height="2.622221128608924in"}

**Figure 6.5: GRU multiple people - learning curve**

 As can be seen, the cross-entropy loss function of the trained data
 decreases together with the validation loss function -- along the
 advancing epochs. This decrease goes on until saturating at about the
 same epoch. The trend of both validation and training is similar,
 implying that there is correlation between the training and validation
 -- whenever the network learns the training dataset the validation
 improves. The step detection accuracy received when comparing the
 output to the label on the multiple people dataset at this experiment
 was **94.6%**.

 **6.2.3** **t-SNE test set representation**

 In order to get a sense of the neural network separation process, we
 extract a t-SNE representation of each layer of the GRU network. It is
 assumed that there is a correlation between the classes' separation
 shown after the t-SNE representation, and the accuracy of the NN.
 Shown below, is the representation of the GRU layer output:

![](media/image28.png)

**Figure 6.6: t-SNE representation of the GRU layer output**

 Figure 6.7 display the fully connected layers outputs:

![](media/image29.png)

![](media/image30.png)

**Figure 6.7: : t-SNE representation of the fully-connected layers
outputs**

 Figure 6.8 displays output layer representation of the GRU and the CNN
 networks:

![](media/image31.png)
**Figure 6.8: t-SNE representation of multi persons dataset - CNN1D
(left) GRU (right)**

 We can deduct from these t-SNE representations that there is an
 interference among the classes features distributions, which causes
 the networks to separate imperfectly. This is because the features
 distributions are made of mixture of different persons distributions
 (a full separation was received with one person). Although a full
 separation wasn't achieved with this specific experiment of multi-
 persons dataset, the accuracy is still quite high (94%), as the
 majority of the data was indeed separated.

 **6.2.4** **Experiments comparison**

 The NN results achieved different results when trained and tested with
 different groups of people. Denote, that the hyperparameters were not
 adjusted between each experiment, and thus the results could
 theoretically be improved. Another result that rises from the
 comparison, is that the GRU achieved better results than the CNN, but
 only by a small difference.

 **7** **Result analysis**

 This chapter describes the analyses of the results achieved.

 **7.1** **Single Person**

 As shown in Figure 6.1 and in Figure 6.2 -- Both NN architectures
 enabled a perfect step detection when trained and tested with a single
 person dataset. Although the CNN learned faster, it could be seen that
 both architectures were able to handle a time series of recorded (and
 not simulated) data. Although the networks' internal architecture
 differs a lot, they still did not struggle with the simple task of
 detecting a step of a single person. Nonetheless, they were able to
 learn the IMU sensed data that is related to a step of any specific
 person.

 **7.2** **Multiple people**

 As shown in Figure 6.4 and in Figure 6.5-- Both NN architectures
 enabled a reliable step detection when trained and tested with a
 multiple people datasets. The similar accuracy received by both NN in
 each experiment indicates that both architecture have about the same
 capability.

 As the results differ between each experiment and the hyperparameters
 were not adjusted - it could be inferred that a subset of the data is
 not a reliable representation of the entire dataset. Despite this
 being said, the NN managed to result with a relatively high accuracy
 results. This means that different people walk in a similar pattern in
 general, a phenomena the neural networks capture.

 Another major challenge was computational limitations (which have been
 discussed throughout this work), which might have prevented the neural
 network to generalize, due to the relatively small (and non-diverse)
 dataset it was exposed to.

 **8** **Conclusions**

 This work aimed to solve the step detection problem. This task was
 addressed by training two different types of neural networks -- CNN
 and GRU, to sense whether or not a step occurred, by analyzing a given
 IMU timeseries data. In is important to denote that the work has led
 us to research and ask ourselves some interesting (and even
 philosophical) questions such as: What is a step? When does it start
 or end? What is defined as "not step"? and many more question. As part
 of that research, we had the chance to meet some interesting people,
 such as Prof. Mattfeld, that advised us and helped us figure our way
 through our problem.

 The use of recorded data from an actual IMU, instead of tackling the
 problem with simulated data, had raised many challenges -- as the
 recorded data inherently includes many different types of errors --
 that has been vastly discussed throughout this report. Although
 challenging, by constructing a suitable architecture and by tuning the
 hyperparameters -- high detection accuracy was achieved and
 practically enabled a trusted step detector.

 The main results are that both the CNN1D and GRU neural networks are
 suitable to tackle a noised time series data, and by tuning them in
 the right manner -- could result with a reliable step detector. It was
 interesting to note that the received accuracies were similar to both
 architectures, which are designed in a very different manner. It was
 also clear that as expected - the mission of step detection from a
 multi-people dataset was a more difficult challenge compared to a
 single person dataset. Together with that, it was found that there is
 a "general" walking pattern that characterizes many different people
 and thus a step detection from an IMU reading is a suitable task for a
 NN.

 This work could (and should) be further investigated in many different
 ways, for example: step detection by using another sensor location
 (for example wrist), construct and train a network that could identify
 between steps of different legs and identify if no step occurred,
 train the NN on different type of walks, train the step detector on a
 4 legged robots, input the NN only some measurement and many more.

 **Works Cited**

 \[1\] ,. E. J. a. A. H. Ryan Mattfeld, \"A New Dataset for Evaluating
 Pedometer,\" 2017.

 \[2\] S. Prince, Understanding Deep Learning, The MIT Press, 2023.

 \[3\] N. Shlezinger, \"Introduction to deep learning lectures,\" 2022.

\[4\] B. Lin, \"Machine Learning and Pedometers: An Integration-Based
Convolutional

 Neural Network for Step Counting and Detection,\" 2020.

 **9** **Appendix**\
 **9.1** **Shimmer3 Datasheet**

![](media/image32.png)
