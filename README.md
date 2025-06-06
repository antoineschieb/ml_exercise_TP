# Author

Antoine Schieb

# Please note

All of the training has been done on kaggle servers, which is why it was easier for me to work on a single notebook. The notebook is called 'main.ipynb' (and it used to be called 'explore.ipynb' if you look at earlier versions in the commit history).


# Methodology

Here is a rough overview of how I approached this problem chronologically. This same overview can be observed in the commit history. 

1 - Explore the data: 
    I immediately recognized the CelebA dataset. I also quickly noticed that the majority class was 1 (88%). 
    I also displayed a few examples for each class. I quickly noticed that the feature to predict had to do with the person wearing a hat and/or glasses, and/or probably a combination of other feature(s) (?) which I couldn't make out myself.

2 - I ran a first training run naively, with the regular cross entropy loss, which happened to be quite unsuccessful due to the highly unbalanced data. 

3- Started using FocalLoss and saw immediate improvements, and I was convinced that this would be the way to go, so I then performed a quick grid search (by hand) to optimize the main hyperparameters (learning rate, number of epochs, alpha and gamma). I used a smaller subset of the data to run the grid search (20k images train, 5k val). Interestingly, calibrating my model didn't offer any improvement. The best threshold for minimizing HTER was always around 0.50. 

4- After finding a good set of hyperparameters, I already had a decent HTER (8%-10% depending on the runs and the data split). I then displayed the examples that the model misclassified. I saw that most of the time, the model would make mistakes that even I would do (since I do not know how the feature has been built) such as classify people with a hat or glasses as 0, when they were actually 1 according to train_labels.txt. I really wasn't sure why these examples were classified as 1, since they were all very similar to the 0 class, so I figured that the model & I may be missing a small but important feature (because I know celebA has a lot of obscure features which aren't always obvious). I also noticed that the same samples were often misclassified between runs.

5- My idea was to apply weight to the examples that were hard to classify. But unlike FocalLoss which uses the model's confidence to judge if a sample is hard to classify, I wanted to use knowledge from previous runs. So I divided the dataset in 5 folds. I trained 4 models using 4-fold CV on the first 4 folds, each time using the validation fold to generate the "hard_to_classify" binary labels. If the model misclassified an image, this label would be set to 1; Otherwise 0.
I then used the 5th fold as a test fold to validate this technique, changing the weight of hard examples to see how well it would help my model. It honestly did not help too much. It may even have slightly hurt the performance a little bit (9-11% instead of the initial 8-10%). I started worrying that the labels given had some noise in them, which would explain why the same samples were mysteriously misclassified... (in which case weighting hard examples would be quite catastrophic) 

But I still wanted to keep the method in my solution for the sake of this exercise, because I thought it was interesting to implement, and maybe it would give different results from the other candidates. 

6 - So for the final training I did 5-fold CV to generate my "hard_to_classify" labels, this time on the whole dataset. Then I trained my model on the whole dataset using a weight a 3 for hard examples, and 1 for easy examples, and with the same hyperparameters as before. Finally, I used the model to generate predictions on the provided test set.


# Some more methods to explore...

- Data augmentation: I didn't feel the need to apply data augmentation here because we already had a very large and diverse dataset. But it may well help the performance. 
- Ensembling methods: From my experience, these are often a "quick win" in problems like these, so it would probably be worth a shot. But they're not too interesting to implement.
- Trying out other encoders: I quickly put all my money on efficientnet b0 trained on Imagenet 1K because it's a very versatile model that has proven to be able to adapt to many tasks in computer vision. But it could be worth looking at other models, including some smaller ones because the images are very small (64x64), or even some models specifically pretrained for face recognition tasks.
