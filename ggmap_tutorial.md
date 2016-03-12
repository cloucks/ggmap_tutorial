#ggmap_tutorial

 - **Authors**: Catrina Loucks
 - **Research field**: Genetics
 - **Lesson Topic**: In this lesson we will be using ggmap to create maps showing detailed spatial information
 
###Motivation
- Plotting points on maps becomes highly accurate, flexible and reproducible
- ggmap uses the grammar of graphics that underlies the ggplot2 package  

####First make sure you have ggplot2 and ggmap

```{r-get_ggplot2}
#Make sure ggplot2 is installed
if(!require("ggplot2")) {
    install.packages("ggplot2")
    library(ggplot2)
  } 
```

```{r-get_ggmap}
#Make sure ggmap is installed
if(!require("ggmap")) {
    install.packages("ggmap")
    library(ggmap)
  } 
```

 - Currently, Tiffany Timbers and I are looking for the genetic basis of individual phenotypic differences 
 - We are using worms as a model system and investigating 40 strains sampled from various locations around the world

####Then load the data
 
```{r-load_data}
#load and take a look at data
#set working directory to folder containing the worms_strains.csv file for this tutorial
setwd("~/Documents/PhD/Classes/Scientific_programming_study_group/ggmap")
#make an object called worm_strains to hold the table
worm_strains <- read.table("worm_strains.csv", header=TRUE, sep=",")
#take a look at worm_strains, containing four columns (strain name, location sampled, latitude sampled, logitude sampled)
head(worm_strains)
```

####Now find the location of the first strain: AB1

```{r-determining_map_location}
#make an object for the location where AB1 was sampled
AB1_location <- "Adelaide, Australia"
#you can also define the location using the latitude/longitude coordinates (geocode can determine these)
geocode("Adelaide, Australia")
AB1_location <- c(lon=138.6, lat=-34.92862)
#find a map including the location of AB1
AB1_map <- get_map(location=AB1_location, source="stamen", maptype="watercolor", crop=FALSE) 
ggmap(AB1_map)
```

##Challenge 1

Use the ggmap cheatsheet to get a map showing the location of strain JU263
- use a different source for your map
- try making the map black and white
- zoom to the level of the continent

####Now we are going to plot the locations of all our worm strains on a map

```{r-get_europe_map}
#find a map including with the location set as Europe
europe_map <- get_map(location="Europe", source="google", maptype="terrain", color = "bw", zoom=3) 
ggmap(europe_map)
```

```{r-finding_lon/lat}
#first determine latitude/longitude locations for each strain and add them to the worms_strains dataframe
worm_strains$location <- geocode(worm_strains$location)[1]
Error: is.character(location) is not TRUE
#make sure the location column is in the character class
worm_strains$location <- as.character(worm_strains$location)
worm_strains$x <- geocode(worm_strains$location)[1]
worm_strains$y <- geocode(worm_strains$location)[2]
```

```{r-mapping_strains}
ggmap(europe_map) +
geom_point(data=worm_strains, aes(x=x, y=y))
#play around with parameters
ggmap(europe_map) +
geom_point(data=worm_strains, aes(x=x, y=y), alpha=0.3, color="red", size=5)

```

####Now make a plot with tap_number on the x-axis and reversal on the y-axis
- ggplot starts with two arguments: data and aesthetic mapping
 
```{r-make_plot}
#Assign plot to object p 
p <- ggplot(data, aes(tap_number, reversal)) #aes is short for aesthetic mapping
```

####You need to add layers to display graph
 - A layer can be as simple as specifying a geom 
 - Geom is short for geometric object and describes the type of object that is used to display the data 
 
```{r-add_layers}
#you can add layers to the plot with a +
p + geom_point() #geom_point is added to make data a scatterplot
```

####You can also manipulate data directly in ggplot2
 - All strains should respond similarly to the first tap, but we see that the two strains start at different points on the y-axis
```{r-manipulate_data}
#Look at habituation data
data

#We have a column with reversal, which represents the number of worms reversing, and a column with N, which represents the total number of worms looked at during the tap
#To look at the proportion of worms reversing, we need to divide reversal by N
#This can be done directly in ggplot2

p <- ggplot(data, aes(tap_number, reversal/N)) + #Manipulate the y-axis to become reversal/N
  geom_point()
p

#Now both strains show a similar response to the intial tap
```

####Now we can play with aesthetics 
 - Aesthetics are visual properties affecting the way observations are displayed (e.g. colour, size and shape)

```{r-adding_aesthetics}
#You can assign categorial variables a colour or shape and continuous variables to size

p  + geom_point(aes(colour=strain)) #Use aesthetics to colour points in scatterplot according to strain

p + geom_point(aes(shape=strain)) #Different shapes represent the different strains

p + geom_point(aes(colour=tap_number, shape=strain)) #Can string two aesthetics together using commas

p + geom_point(aes(colour=tap_number, shape=strain, size=7)) #Can also play around with size
```
  
  
####You can also experiment with geoms to get different kinds of plots 
- (e.g. scatterplot=geom_point(), line plot = geom_line(), and boxplot=geom_boxplot())
```{r-varying_geom}
p + geom_point(aes(colour=strain))

p + geom_point(aes(colour=strain)) + #Can add other layers with the +
  geom_line() #Make graph a line plot

p + geom_point(aes(colour=strain)) +
  geom_line(aes(group=strain)) #Add group in aesthetics to make lines attach points from the same strain

p + geom_point(aes(colour=strain)) +
  geom_line(aes(colour=strain)) #Changing group to colour in aesthetics to make the lines the same colour as the points
```


##Challenge 1

Make a boxplot of strain on x-axis (categorical variable) and N (number of animals assessed) on y-axis to determine how many animals were tested

 - hint: use geom_boxplot()


####You can also add error bars to the graph
```{r-error_bars}
p + geom_point(aes(colour=strain)) +
  geom_line(aes(colour=strain)) +
  geom_errorbar(aes(ymin=conf_int_lower, ymax=conf_int_upper, colour=strain))
```

####Next, we can make labels (title, axes and legend)
```{r-making_labels}
p + geom_point(aes(colour=strain)) +
  geom_line(aes(colour=strain)) +
  geom_errorbar(aes(ymin=conf_int_lower, ymax=conf_int_upper, colour=strain)) +
  ggtitle("Habituation probability") + #Add title
  xlab("Tap number") + #Add x-label
  ylab("Proportion reversing") + #Add y-label
  scale_colour_hue("type of worm", #Change legend title
    breaks = c("mutant", "N2"), #Determine what strains are on the legend
    labels = c("mutant", "wild-type")) #Determine the names of the legend strains
```

####We can also decide what values we want on our axes
```{r-changing_axes}
p + geom_point(aes(colour=strain)) +
  geom_line(aes(colour=strain)) +
  geom_errorbar(aes(ymin=conf_int_lower, ymax=conf_int_upper, colour=strain)) +
  ggtitle("Habituation probability") + #Add title
  xlab("Tap number") + #Add x-label
  ylab("Proportion reversing") + #Add y-label
  scale_colour_hue("type of worm", #Change legend title
    breaks = c("mutant", "N2"), #Determine what strains are on the legend
    labels = c("mutant", "wild-type")) +
  scale_x_continuous(limits=c(0,31), breaks=c(0,5,10,15,20,25,30)) + #Change limits and spacing in x-axis
  scale_y_continuous(limits=c(0.0,1.0), breaks=c(0.0,0.20,0.40,0.60,0.8,1.0)) #Change limits and spacing in y-axis
```

##Challenge 2

Make a plot with tap_number on the y-axis and proportion reversing on the y-axis that only displays the last 10 taps

 - hint: change limits of axis


####Finally we can play around with the style of the graph
```{r-change_style}
p + geom_point(aes(colour=strain)) +
  geom_line(aes(colour=strain)) +
  geom_errorbar(aes(ymin=conf_int_lower, ymax=conf_int_upper, colour=strain)) +
  ggtitle("Habituation probability") + #Add title
  xlab("Tap number") + #Add x-label
  ylab("Proportion reversing") + #Add y-label
  scale_colour_hue("type of worm", #Change legend title
    breaks = c("mutant", "N2"), #Determine what strains are on the legend
    labels = c("mutant", "wild-type")) +
  scale_x_continuous(limits=c(0,31), breaks=c(0,5,10,15,20,25,30)) + #Change limits and spacing in x-axis
  scale_y_continuous(limits=c(0.0,1.0), breaks=c(0.0,0.20,0.40,0.60,0.8,1.0)) + #Change limits and spacing in y-axis 
  theme_classic() + #Gives a nice white background without gridlines
  theme(legend.title=element_blank()) #Removes legend title (can override the previous manipulation of the legend)
```
  
Save plot
```{r-save_plot}
p + geom_point(aes(colour=strain)) +
  geom_line(aes(colour=strain)) +
  geom_errorbar(aes(ymin=conf_int_lower, ymax=conf_int_upper, colour=strain)) +
  ggtitle("Habituation probability") + #Add title
  xlab("Tap number") + #Add x-label
  ylab("Proportion reversing") + #Add y-label
  scale_colour_hue("type of worm", #Change legend title
    breaks = c("mutant", "N2"), #Determine what strains are on the legend
    labels = c("mutant", "wild-type")) +
  scale_x_continuous(limits=c(0,31), breaks=c(0,5,10,15,20,25,30)) + #Change limits and spacing in x-axis
  scale_y_continuous(limits=c(0.0,1.0), breaks=c(0.0,0.20,0.40,0.60,0.8,1.0)) + #Change limits and spacing in y-axis 
  theme_classic() +
  theme(legend.title=element_blank()) #Removes legend title
#ggsave("final_plot.pdf", width=9, height=6)
```

###Answer to challenge 1
```{r-determining_map_location}
#make an object for the location where JU263 was sampled
JU263_location <- "Le Blanc, France"
#find a map including the location of JU263
JU263_map <- get_map(location=JU263_location, source="google", maptype="satellite", color = "bw", zoom=3) 
ggmap(JU263_map)
```

###Answer to challenge 2



