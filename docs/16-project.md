# Project analysis

We've spent the last 6 months giving you the skills you need to be able to deal with your own data. Now's the time to show us what you've learned. In this chapter we're going to describe the steps you will need to go through when analysing your data but, aside from a few lines that will help you deal with the questionnaire data and a couple of things you haven't done before we're not going to give you any code solutions. 

For everything else, you have done it before, so use this book to help you. Remember, you don't need to write the code from memory, you just need to find the relevant examples and then copy and paste and change what needs changing to make it work for you.

We suggest that you problem-solve the code as a group, however, make sure that you all have a separate copy of the final working code. You can book into GTA sessions in week 9 and 10 to help you as well.

## Step 1: Import and initial data wrangling

To help get you on your way, we've done a bit of the initial wrangling for you. Download the data files from Moodle into your working directory, create a new Markdown, then run all the below code without changing anything. 


```r
library(tidyverse)

# read in data but skip rows 2 and 3
col_names <- names(read_csv("big5_data.csv", n_max = 0))
dat <- read_csv("big5_data.csv", col_names = col_names, skip = 3) %>%
  filter(Progress == 100, # only keep data from participants who progressed to end of survey
         DistributionChannel != "preview")  # Remove Emily's preview data

# read in scoring and codebook data
scoring <- read_csv("scoring.csv")
codebook <- read_csv("code_book.csv")

# create dataset of just big 5 data then join with codebook and scoring
big5 <- select(dat, ResponseId, Q4_1:Q4_60) %>%
  pivot_longer(cols = Q4_1:Q4_60, names_to = "item", values_to = "response") %>%
  inner_join(codebook, by = "item") %>%
  inner_join(scoring, by = c("response", "direction"))

# create dataset that just has demographic questions

demographic <- select(dat, ResponseId, 
                      "age" = Q6, 
                      "gender" = Q7, 
                      "sexual_orientation" = Q8, 
                      "siblings" = Q9, 
                      "birth_order" = Q10, 
                      "neurodivergent" = Q11,
                      "team_name" = Q14) %>%
  mutate(gender = factor(gender, labels = c("Man", "Woman", "Non-binary", "Prefer not to say")),
         sexual_orientation = factor(sexual_orientation, labels = c("Straight/ Heterosexual", "Bisexual/pansexual", "Gay or Lesbian", "Asexual", "Other", "Prefer not to say")),
         birth_order = factor(birth_order),
         neurodivergent = factor(neurodivergent, labels = c("Neurotypical", "Diagnosed", "Seeking diagnosis", "Not seeking diagnosis", "Prefer not to say")),
         team_name = factor(team_name))

# get rid of objects we don't need
rm(dat, scoring, codebook)
```

## Step 2: BFI-2 scores

First, let's deal with the Big 5 scores and just focus on the whole dataset rather than your team data.

Create an object named `trait_dat` (where `trait` is the name of the Big 5 trait you are using as your DV) that just has the data from the trait you are interested in. Your starting dataset should be `big5`.

Hint: `filter()`

Then, use `trait_dat` to calculate the average trait `score` for each participant (`ResponseId`). Store these scores in an object named `trait_scores`.

Hint: `group_by()` and then `summarise()`.

## Step 3: IV

Now, let's deal with the demographic information you're using as your IV. We want to wrangle the dataset according to the decisions you made in your registered report. You can save each of the below steps to a different object, or you can use the `%>%`. However you do it, make the final obkect named `my_demo`.

1. First, if you have exclusion criteria, filter out any participants you don't want in your dataset (e.g., if you have an age limit, or only don't want only children). You may also want to remove people who answered "Prefer not to say" for your IV at this point.
2. Then, select to keep just the columns that contain the participant's id, age, gender, team name, and your IV (assuming it isn't gender, in which case you only need those two). 
3. Then, create any new category groups as required. For example, you might want to collapse sexual orientation into two categories, "heterosexual" and "queer". This is slightly complicated so we'll give you example code - you can adapt the code to collapse other categories like the neurodivergent or birth order options. Be very careful with the spelling, it needs to be exact for `recode` to work.
4. Finally, add `drop_na(IV)` and `droplevels()` to your code (see below) where IV is the variable that you're using as your IV. This will get rid of anyone who didn't respond to your IV question (because obviously you can't include them in your analysis). `droplevels` removes any factors that don't have any data points (if you don't do this your plots will still show the prefer not to say category even if there's no data in this category).


```r
my_demo <- data_from_step1_and_step2 %>%
  mutate(sexual_orientation_recoded = recode(sexual_orientation, 
                                             "Straight/ Heterosexual" = "Heterosexual",
                                             "Gay or Lesbian" = "Queer",
                                             "Bisexual/pansexual" = "Queer",
                                             "Asexual" = "Queer",
                                             "Other" = "Queer")) %>%
  drop_na(sexual_orientation) %>% # put your IV variable in here
  droplevels()
```

## Step 4: Joining it all together

Getting there.

Now, create a dataset named `all_dat` that joins together `my_demo` and `trait_scores` so that you have a single dataset that contains all the data you need from participants who are present in both datasets.

Hint: `inner_join()`

Now on to the analysis (the worst part is over).

## Step 5: Sample information

Count the number of participants in the data set. This is the overall class sample size. You haven't done this very often so we'll give you the code.


```r
all_dat %>%
  count()
```

Count the number of people of each gender in `all_dat`. You can also create a column of percentages. You haven't done this before so we'll also give you the code.


```r
all_dat %>%
  count(gender) %>%
  mutate(percent = n/sum(n)*100)
```

Calculate the mean age and SD of the full sample. If there is missing data, you may need to use `na.rm = TRUE`.

Hint: `summarise()`

Count how many participants are in each of your IV groups, e.g., how many first born and late born, or how many neurodivergent and neurotypical. Remember to use the recoded variable if you collapsed the categories. You may also wish to calculate percentages.

## Step 6: Group differences in scores

Calculate the mean and SD trait score for each of your groups. If there is missing data, you may need to use `na.rm = TRUE`.

Hint: `group_by()` then `summarise()`

## Step 7: Visualisations

Create a bar chart of means for your DV by IV with error bars that display standard error (`mean_se`). 

Create a violin-boxplot for your DV by IV. 

Ensure that your visualisations are appropriately labelled and try to make them look visually appealing.

## Step 8: Team data

Almost done. Now, create a dataset that just contains the data from your team. Be very careful about the spelling of your team name, there is a space before and after the colon and it must be exactly as it is in the dataset to work including all capital letters. You might want to open up the data file to check the exact spelling. Here's an example.


```r
team_dat <- all_dat %>%
  filter(team_name == "PR01 : Pavlov's Puppies")
```

Then repeat Steps 5 - 7 but use `team_dat` instead of `all_dat`. You should be able to reuse all of code, and just plug in the different dataset. 

## Finished

And you're done! I wish that I could adequately convey how impressive what you've just done is. I wish that I could show you how amazing your skills are compared to most other psychology undergraduates in the UK and across the world. 
