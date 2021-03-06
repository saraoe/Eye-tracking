``` r
Samples <- read_csv("Exported_EyeLink_data/Cleaned/Samples_merged_Fabio.csv") %>% 
  mutate(GazeY = 1051-GazeY, Fix_MeanY = 1051-Fix_MeanY) %>% #As the y-axis is flipped in Eyelink (1051 because you want to keep the zero!)
  filter(Time<=41202) #There are so few datapoints with the incorrect time that we will just filter them out
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_double(),
    ##   ParticipantID = col_character(),
    ##   ParticipantGender = col_character(),
    ##   EyeTracked = col_character(),
    ##   Task = col_character(),
    ##   ForagingType = col_character(),
    ##   Stimulus = col_character(),
    ##   Video = col_logical(),
    ##   Sac_Blink = col_logical(),
    ##   Sac_Direction = col_character()
    ## )

    ## See spec(...) for full column specifications.

Sanity checks
-------------

### Check distribution of fixations

Let's start with density plots

``` r
# before doing this we must make a summary dataset

Fix <- Samples[!is.na(Samples$FixationNo),] %>% # remember to remove NAs 
  group_by(ParticipantID, Trial, FixationNo) %>% 
  summarize( 
            Fix_Duration = Fix_Duration[1],
            Task = Task[1], ParticipantGender = ParticipantGender[1]) %>% 
  group_by(ParticipantID, Trial) %>% 
  summarize(Fix_Number = max(FixationNo),
            Fix_Duration = mean(Fix_Duration),
            Task = Task[1], ParticipantGender = ParticipantGender[1])

# plot density of fixation number
ggplot(Fix, aes(Fix_Number, color = ParticipantID)) + geom_density() + facet_wrap(.~Task)
```

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20fixations-1.png)

``` r
# plot density of fixation duration
ggplot(Fix, aes(Fix_Duration, color = ParticipantID)) + geom_density() + facet_wrap(.~Task)
```

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20fixations-2.png)

We can also use histograms:

``` r
# plot density of fixation number
ggplot(Fix, aes(Fix_Number, fill = ParticipantGender)) + geom_histogram() + facet_wrap(.~Task)
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20fixations%20histograms-1.png)

``` r
# plot density of fixation duration
ggplot(Fix, aes(Fix_Duration, fill = ParticipantGender)) + geom_histogram() + facet_wrap(.~Task)
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20fixations%20histograms-2.png)

### Check distribution of saccades

### - notice anything interesting about the number of saccades?

``` r
## Check distribution of saccades

# before doing this we must make a summary dataset
Sac <- Samples[!is.na(Samples$SaccadeNo),] %>% # remember to remove NAs 
  group_by(ParticipantID, Trial, SaccadeNo) %>% 
  summarize(Sac_Duration = Sac_Duration[1],
            Sac_Amplitude = Sac_Amplitude[1],
            Task = Task[1], ParticipantGender = ParticipantGender[1]) %>% 
  group_by(ParticipantID, Trial) %>% 
  summarize(Sac_Number = max(SaccadeNo),
            Sac_Duration = mean(Sac_Duration),
            Sac_Amplitude = mean(Sac_Amplitude),
            Task = Task[1], ParticipantGender = ParticipantGender[1])


# plot density of saccade number
ggplot(Sac, aes(Sac_Number, color = ParticipantID)) + geom_density() + facet_wrap(.~Task)
```

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20saccades-1.png)

``` r
# plot density of saccade duration
ggplot(Sac, aes(Sac_Duration, color = ParticipantID)) + geom_density() + facet_wrap(.~Task)
```

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20saccades-2.png)

``` r
# plot density of saccade amplitude
ggplot(Sac, aes(Sac_Amplitude, color = ParticipantID)) + geom_density() + facet_wrap(.~Task)
```

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20saccades-3.png)

``` r
# plot density of saccade number by gender
ggplot(Sac, aes(Sac_Number, color = ParticipantGender)) + geom_density() + facet_wrap(.~Task)
```

![](Data_visualizations_files/figure-markdown_github/sanity%20checks%20saccades-4.png)

### Remove all the data points that fall outside of the screen coordinates (1680, 1050)

``` r
# before...
plot(density(Samples$GazeX, na.rm = TRUE))
```

![](Data_visualizations_files/figure-markdown_github/remove%20artefacts-1.png)

``` r
plot(density(Samples$GazeY, na.rm = TRUE))
```

![](Data_visualizations_files/figure-markdown_github/remove%20artefacts-2.png)

``` r
Samples <- Samples %>% filter(GazeX >= 0 & GazeX <= 1680 & GazeY >= 0 & GazeY <= 1050)

# ...and after
plot(density(Samples$GazeX, na.rm = TRUE))
```

![](Data_visualizations_files/figure-markdown_github/remove%20artefacts-3.png)

``` r
plot(density(Samples$GazeY, na.rm = TRUE))
```

![](Data_visualizations_files/figure-markdown_github/remove%20artefacts-4.png)

### Check distribution of pupil sizes

``` r
# before doing this we must make a summary dataset
Pup <- Samples[!is.na(Samples$PupilSize),] %>% # remember to remove NAs 
  group_by(ParticipantID, Trial) %>% 
  summarize(PupilSize = mean(PupilSize), 
            Task = Task[1], ParticipantGender = ParticipantGender[1])

# plot density of pupil sizes
ggplot(Pup, aes(PupilSize, color = ParticipantID)) + geom_density() + facet_wrap(.~Task)
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-1-1.png)

Visualizations
--------------

### Scanpath

``` r
## Here I am making the scanpath for one participant in one trial
x = subset(Samples, ParticipantID ==    'F7_2' & Trial == 10)
x$FixationNo <- as.factor(x$FixationNo)

## Let's make a summary dataset
Fix <- x[!is.na(x$FixationNo),] %>% 
  group_by(FixationNo) %>% # since I only have one participant and one trial
  summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1]) %>% 
  filter(Duration>=300) # only keep fixations > 300 ms

img <- jpeg::readJPEG('stimuli_Foraging/space_capsules.jpg')  
img <- grid::rasterGrob(img, width=unit(1, "npc"), height = unit(1,"npc"),
                        interpolate = FALSE)

ggplot(Fix, aes(MeanX, MeanY, color = Fix$FixationNo)) + 
  annotation_custom(img, xmin = 0, xmax = 1680, ymin = 0, ymax = 1050) +
  geom_path(color = "black") +
  geom_point(size = Fix$Duration*.02, alpha = .8) +
  ggrepel::geom_text_repel(aes(label = Fix$Duration), size = 3, color = "white") +
  xlim(0,1680) + ylim(0,1050)
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-2-1.png)

Scanpath for Social engagement experiment

``` r
## Here I am making the scanpath for one participant in one trial
x = subset(Samples, ParticipantID ==    'F1' & Trial == 4)
x$FixationNo <- as.factor(x$FixationNo)

## Let's make a summary dataset
Fix <- x[!is.na(x$FixationNo),] %>% 
  group_by(FixationNo) %>% # since I only have one participant and one trial
  summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1]) %>% 
  filter(Duration>=300) # only keep fixations > 300 ms

ggplot(Fix, aes(MeanX, MeanY, color = Fix$FixationNo)) + 
  geom_path(color = "black") +
  geom_point(size = Fix$Duration*.02, alpha = .8) +
  ggrepel::geom_text_repel(aes(label = Fix$Duration), size = 3, color = "black") +
  xlim(0,1680) + ylim(0,1050)
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-3-1.png)

Using a for-loop, make a scanpath for each participant in the Foraging experiment. Use facets to plot the 10 trials separately for each participant. Use these plots as diagnostic tools in order to answer the following questions:

1.  Do the data look reasonable and of good quality? Do we have any issues?
2.  Can we differentiate between the two conditions (Count and Search) only by looking at the scanpaths?
3.  Can we spot the trials in which the participants found the star?

``` r
library(pacman)
p_load(gridExtra)

foraging <- Samples[Samples$Task == 'Foraging',]

###Til Sara Vestergaard
participants <- as.character(levels(as.factor(foraging$ParticipantID))) #Character string with all participantID

for(i in 1:length(participants)){
  for(t in levels(as.factor(foraging$Trial)))
    x = subset(foraging, ParticipantID ==   participants[i], Trial = t) #Choosing only one participant and one trial
    
    for(p in levels(as.factor(x$Stimulus))){
      Fix <- x[!is.na(x$FixationNo) & x$Stimulus == p,] %>% 
      group_by(FixationNo) %>% # since I only have one participant and one trial
      summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1],
                Stimulus = Stimulus[1], Trial = Trial[1])%>% 
      filter(Duration>=300) # only keep fixations > 300 ms
      
      picpath <- paste('stimuli_Foraging/',p, sep = "" )
      img <- jpeg::readJPEG(picpath)  
      img <- grid::rasterGrob(img, width=unit(1, "npc"), height = unit(1,"npc"),
                        interpolate = FALSE)
      
      plotname <- paste("Participant: ", participants[i], " Trial: ", t, " Picture: ", p, sep = "")
      
      great_plot <- ggplot(Fix, aes(MeanX, MeanY, color = Fix$FixationNo)) + 
      annotation_custom(img, xmin = 0, xmax = 1680, ymin = 0, ymax = 1050) +
      geom_path(color = "black") +
      geom_point(size = Fix$Duration*.02, alpha = .8) +
      ggrepel::geom_text_repel(aes(label = Fix$Duration), size = 3, color = "white") +
      xlim(0,1680) + ylim(0,1050) + ggtitle(plotname)
      
      print(great_plot) #Print plot so we don't overwrite each plot
    }
    ## could use patchwork function to make the plots look nicer
}
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-1.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-2.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-3.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-4.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-5.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-6.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-7.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-8.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-9.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-10.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-11.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-12.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-13.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-14.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-15.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-16.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-17.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-18.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-19.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-20.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-21.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-22.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-23.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-24.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-25.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-26.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-27.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-28.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-29.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-30.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-31.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-32.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-33.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-34.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-35.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-36.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-37.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-38.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-39.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-40.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-41.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-42.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-43.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-44.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-45.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-46.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-47.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-48.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-49.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-50.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-51.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-52.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-53.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-54.png)![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-4-55.png)

### Heatmap

``` r
## Here is a palette of heatmap-friendly colors
heat_colors <- colorRampPalette(
  c(
    "#00007F",
    "blue",
    "#007FFF",
    "cyan",
    "#7FFF7F",
    "yellow",
    "#FF7F00",
    "red",
    "#7F0000"
  )
)
```

``` r
## Here I am making the scanpath for one participant in one trial
x = subset(Samples, ParticipantID ==    'F7_2' & Trial == 1)

## Let's make a summary dataset
Fix <- x[!is.na(x$FixationNo),] %>% 
  group_by(FixationNo) %>% # since I only have one participant and one trial
  summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1]) %>% 
  filter(Duration>=300) # only keep fixations > 300 ms

img <- jpeg::readJPEG('stimuli_Foraging/sheep.jpg')  
img <- grid::rasterGrob(img, width=unit(1, "npc"), height = unit(1,"npc"),
                        interpolate = FALSE)

ggplot(Fix, aes(MeanX, MeanY, color = Fix$FixationNo)) + 
  annotation_custom(img, xmin = 0, xmax = 1680, ymin = 0, ymax = 1050) +
  stat_density2d(geom = "raster", aes(fill = ..density.., alpha = sqrt(sqrt(..density..))), contour = FALSE, n = 1000) + 
  scale_fill_gradientn(colours = heat_colors(10), trans="sqrt") +
  scale_alpha(range = c(0.1, 0.6)) +
  xlim(0,1680) + ylim(0,1050) +
  theme(legend.position = "none")
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-6-1.png)

Excercise: Make a cumulative heatmap for all participants in the Foraging experiment looking at the 'penguins.jpg' image and facet the graph by Foraging Type (Search vs. Count). What do you notice?

``` r
## Subset data for only penguins trials
penguin = subset(Samples, Stimulus == 'penguins.jpg')

## Let's make a summary dataset
Fix_pen <- penguin[!is.na(x$FixationNo),] %>% 
  group_by(ParticipantID, FixationNo) %>% # since there is only one trial per participant with penguins
  summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1], ForagingType = ForagingType[1]) %>% 
  filter(Duration>=300) # only keep fixations > 300 ms

## Load penguins picture
img <- jpeg::readJPEG('stimuli_Foraging/penguins.jpg')  
img <- grid::rasterGrob(img, width=unit(1, "npc"), height = unit(1,"npc"),
                        interpolate = FALSE)

## Same plot as above, expect facet_wrap, as we have two conditions with the penguins
ggplot(Fix_pen, aes(MeanX, MeanY, color = Fix_pen$FixationNo)) + 
  annotation_custom(img, xmin = 0, xmax = 1680, ymin = 0, ymax = 1050) +
  stat_density2d(geom = "raster", aes(fill = ..density.., alpha = sqrt(sqrt(..density..))), contour = FALSE, n = 1000) + 
  scale_fill_gradientn(colours = heat_colors(10), trans="sqrt") +
  scale_alpha(range = c(0.1, 0.6)) +
  xlim(0,1680) + ylim(0,1050) +
  theme(legend.position = "none")+
  facet_wrap(.~ForagingType)
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-7-1.png)

### AOIs

``` r
## Define an AOI for the black sheep
AOI = c(720, 930, 50, 330)
      # xmin xmax ymin ymax
```

``` r
## Let's make a summary dataset
Fix <- Samples[!is.na(Samples$FixationNo),] %>% 
  group_by(ParticipantID, Trial, FixationNo) %>% # since I only have one participant and one trial
  summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1]) %>% 
  filter(Duration>=300 & # only keep fixations > 300 ms
         MeanX >= AOI[1] & MeanX <= AOI[2] & MeanY >= AOI[3] & MeanY <= AOI[4])

img <- jpeg::readJPEG('stimuli_Foraging/sheep.jpg')  
img <- grid::rasterGrob(img, width=unit(1, "npc"), height = unit(1,"npc"),
                        interpolate = FALSE)

ggplot(Fix, aes(MeanX, MeanY, color = Fix$FixationNo)) + 
  annotation_custom(img, xmin = 0, xmax = 1680, ymin = 0, ymax = 1050) +
  # this line draws the rectangle
  geom_rect(xmin=AOI[1], xmax=AOI[2], ymin=AOI[3], ymax=AOI[4], fill = NA, size = 1, color = 'red') +
  stat_density2d(geom = "raster", aes(fill = ..density.., alpha = sqrt(sqrt(..density..))), contour = FALSE, n = 1000) + 
  scale_fill_gradientn(colours = heat_colors(10), trans="sqrt") +
  scale_alpha(range = c(0.1, 0.6)) +
  xlim(0,1680) + ylim(0,1050) +
  theme(legend.position = "none")
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-9-1.png)

Excercise: Make a cumulative heatmap for all participants in the Foraging experiment looking at the 'dolphins.jpg' image and facet the graph by Foraging Type (Search vs. Count) *after having created an AOI*. What do you notice?

``` r
## Making a subset with only image puinguin
x = subset(Samples, Stimulus == 'dolphins.jpg' & Task == 'Foraging')

# define area of interest
AOI = c(459, 950, 250, 500)
      # xmin xmax ymin ymax

## Let's make a summary dataset
Fix <- x[!is.na(x$FixationNo),] %>%   # only keep what fixations and remove saccades
  group_by(ParticipantID, Trial, FixationNo) %>%
  summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1], ForagingType = ForagingType[1]) %>% 
  filter(Duration>=300 & # only keep fixations > 300 ms
         MeanX >= AOI[1] & MeanX <= AOI[2] & MeanY >= AOI[3] & MeanY <= AOI[4])

img <- jpeg::readJPEG('stimuli_Foraging/dolphins.jpg')  
img <- grid::rasterGrob(img, width=unit(1, "npc"), height = unit(1,"npc"),
                        interpolate = FALSE)

ggplot(Fix, aes(MeanX, MeanY, color = Fix$FixationNo)) + 
  annotation_custom(img, xmin = 0, xmax = 1680, ymin = 0, ymax = 1050) +
  # this line draws the rectangle
  geom_rect(xmin=AOI[1], xmax=AOI[2], ymin=AOI[3], ymax=AOI[4], fill = NA, size = 1, color = 'red') +
  stat_density2d(geom = "raster", aes(fill = ..density.., alpha = sqrt(sqrt(..density..))), contour = FALSE, n = 1000) + 
  scale_fill_gradientn(colours = heat_colors(10), trans="sqrt") +
  scale_alpha(range = c(0.1, 0.6)) +
  xlim(0,1680) + ylim(0,1050) +
  theme(legend.position = "none") +
  facet_wrap(.~ForagingType) # deviding in plot in respect to foraging t
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-10-1.png)

### Growth curves

Growth curves show how proportional looking at one or more specific AOIs changes over time and across participants. Let's start by definining to AOIs:

``` r
## Define an AOI for the black sheep
AOI1 = c(300, 700, 200, 450)
AOI2 = c(600, 1100, 600, 750)
      # xmin xmax ymin ymax
```

Let's make a summary dataset for fixations and filter the fixations that fall within one of the two AOIs. The plot below shows what the two AOIs look like:

``` r
## Let's make a summary dataset
Fix <- Samples[!is.na(Samples$FixationNo),] %>% 
  group_by(ParticipantID, Trial, FixationNo) %>% # since I only have one participant and one trial
  summarize(MeanX = Fix_MeanX[1], MeanY = Fix_MeanY[1], Duration = Fix_Duration[1],
            Stimulus = Stimulus[1]) %>% 
  filter(Duration>=300 & Stimulus=="trees.jpg") %>%
  mutate(InAOI1 = ifelse(MeanX >= AOI1[1] & MeanX <= AOI1[2] & MeanY >= AOI1[3] & MeanY <= AOI1[4], TRUE, FALSE),
         InAOI2 = ifelse(MeanX >= AOI2[1] & MeanX <= AOI2[2] & MeanY >= AOI2[3] & MeanY <= AOI2[4], TRUE, FALSE))

img <- jpeg::readJPEG('stimuli_Foraging/trees.jpg')  
img <- grid::rasterGrob(img, width=unit(1, "npc"), height = unit(1,"npc"),
                        interpolate = FALSE)

ggplot(Fix, aes(MeanX, MeanY, color = Fix$FixationNo)) + 
  annotation_custom(img, xmin = 0, xmax = 1680, ymin = 0, ymax = 1050) +
  # this line draws the rectangle
  geom_rect(xmin=AOI1[1], xmax=AOI1[2], ymin=AOI1[3], ymax=AOI1[4], fill = NA, size = 1, color = 'red') +
  annotate(geom = "label", x = 500, y = 450, label = "AOI1", color = "red") +
  geom_rect(xmin=AOI2[1], xmax=AOI2[2], ymin=AOI2[3], ymax=AOI2[4], fill = NA, size = 1, color = 'blue') +
  annotate(geom = "label", x = 850, y = 750, label = "AOI2", color = "blue") +
  #stat_density2d(geom = "raster", aes(fill = ..density.., alpha = sqrt(sqrt(..density..))), contour = FALSE, n = 1000) + 
  #scale_fill_gradientn(colours = heat_colors(10), trans="sqrt") +
  #scale_alpha(range = c(0.1, 0.6)) +
  xlim(0,1680) + ylim(0,1050) +
  theme(legend.position = "none")
```

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-12-1.png)

Now let's make a new summary dataset where we compute proportions of fixations in either of the two AOIs divided by total number of fixations, and let's plot this proportion using a smoothing function. Do we notice anything interesting?

``` r
Prop <- Fix %>% 
  group_by(FixationNo) %>% 
  summarize(AOI1 = sum(InAOI1 == TRUE)/(length(InAOI1)+length(InAOI2))*100,
            AOI2 = sum(InAOI2 == TRUE)/(length(InAOI1)+length(InAOI2))*100) %>% 
  gather("AOI", "Proportion", AOI1:AOI2)

ggplot(Prop, aes(FixationNo, Proportion, color = AOI)) +
  geom_smooth() + ylim(-10,100)
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-13-1.png)

Exercise: Try adding a third AOI and computing proportional looks to it:

#### Growth curves for pupil size

Here we are going to plot the raw data since we are not interested in distinguishing between fixations and saccades — we just want to know the total change in pupil size across a trial:

*Notice the different scales on the x axis. How do we interpret these results?*

``` r
ggplot(Samples, aes(Time, PupilSize, color = ParticipantGender)) +
  geom_smooth() + facet_wrap(.~Task, scales = "free_x")
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-15-1.png)

``` r
ggplot(Samples, aes(Time, PupilSize, color = ParticipantID)) +
  geom_smooth() + facet_wrap(.~Task, scales = "free_x")
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-15-2.png)

``` r
soc <- subset(Samples, Task == 'SocialEngagement')

ggplot(soc, aes(Time, PupilSize, color = ParticipantGender)) +
  geom_smooth() #+ facet_wrap(.~Task, scales = "free_x")
```

    ## `geom_smooth()` using method = 'gam' and formula 'y ~ s(x, bs = "cs")'

![](Data_visualizations_files/figure-markdown_github/unnamed-chunk-15-3.png)
