# RT-Assignment
Outlined below is the process of using `Shiny` with `Flexdashboard` in `R` to create an interactive dashboard. The steps are ordered chronologically and should be followed as written. Helpful links are also included.  

This project was created as part of the 'Robust Tools Assignment'. 

## Run down of Shiny and Flexdashboard
`Flexdashboard` is a useful way of using `R Markdown` to group and visualise related data as a dashboard. You introduce `Shiny` to a flexdashboard by adding `runtime: shiny` to the YAML header and adding some input controls or reactive expressions. By doing this you turn the static `R Markdown` into an interactive document. What interactive means is that the dashboard enables users to modify parameters (defined by the inputs/ reactive expressions in your `R Markdown`) within the dashboard and immediately view these changes in the output. 

To learn more about `Shiny` and `Flexdashboard` here are some useful resources: 
* [Introduction to interactive documents](https://rmarkdown.rstudio.com/articles_interactive.html)
* [Flexdashboard cheat sheet - YouTube](https://www.youtube.com/watch?v=gkQvhMA24ig)
* [Flexdashboard R cheat sheet](https://rstudio.com/wp-content/uploads/2015/02/shiny-cheatsheet.pdf?fbclid=IwAR0rWb8_KCUbRMkHbtkFwwYhWKINRac9W-MwCzt-_JcwWJ1J1I0hYr7JRFM)
* [Flexdashboard user issues & solutions](https://github.com/rstudio/flexdashboard/issues)
* [Using Shiny](https://bookdown.org/yihui/rmarkdown/shiny.html)
* [Shiny and Flexdashboard](http://jeffgoldsmith.com/p8105_f2017/shiny.html#interactive_boxplot)

## Selecting a dataset
The first step is to find a dataset that you'd like visualise. There are a bunch of resources online that provide free datasets on numerous topics. Some of these include:

* [r-dir](https://r-dir.com/reference/datasets.html)
* [Data World](https://data.world/)
* [FiveThirtyEight](https://fivethirtyeight.com/)
* [Kaggle](https://www.kaggle.com/)

[The Center for Open Science](https://cos.io/) also makes research datasets publicly available. For instance, there is a dataset on the [effect of social media](https://osf.io/n6v8j/) posts on measures of wellbeing including positive affect, negative affect, life satisfaction and subjective wellbeing. This dataset is used in the following code to explain the use of `Shiny` with `Flexdashboard`. 

Once you select a dataset to work on you can begin to decide what you want to do with the data.

## Deciding what to do with the data 
There were many analyses made for the data on the effects of social media on measures of wellbeing. These included: 
* T-tests on each social media condition (luxury items, cute animals, memes) compared to the control
* Pairwise comparisons between each social media condition
* Multiple regression on predictors of wellbeing measures
* Effects of moderating/ mediating variables (like time, age and gender) 

When you have multiple results like this it might be useful to show how the different variables react under different circumstances rather than creating numerous static plots. It is also a useful way for deciding which exploratory analyses to run in psychological research. For this reason the use of `Flexdashboard` and `Shiny` is useful.

So some of the things you could do with the data using `Flexdashboard` is to:
* Clean up and prepare the data for analyses and plots
* Choose variables that you'd like users of the dashboard to be able to change
  * Eg which conditions are shown on the plots and changing the time spent looking at social media posts
* Running t-tests on a subset of the data 
* Plotting these t-test results

The `flexdashboard` that is created below will have two pages. One for the descriptive statistics where users can change the presented Conditions in the boxplot and the time spent looking at a social media post. It will also have an inferential statistics page that will plot static boxplots for comparisons between the Control and each social media condition for all 4 measured outcomes (positive affect, negative affect, life satisfaction and subjective wellbeing).

## Setting up the flexdashboard
[Flexdashboard](https://rmarkdown.rstudio.com/flexdashboard/shiny.html#overview) provides you with an indepth procedure on its use in `R Markdown`. A simple way to start is by selecting a [layout](https://rmarkdown.rstudio.com/flexdashboard/layouts.html) you'd like to work with and including this code in your R Markdown file and then fleshing it out with code for your input and output.
For the social media dataset the following layout was used:

```
---
title: "SOCIAL MEDIA AND WELLBEING"       # Create a title for your dashboard
output: 
  flexdashboard::flex_dashboard:
    orientation: rows                     # Dashboards can be organised in rows or columns*
    vertical_layout: fill                 # You can have the rows/ columns fit on the page or scroll through them
runtime: shiny                            # To make your dashboard interactive add this line of code, to know if this was successful the 'Knit' icon in R Markdown will change to 'Run Document'
---

Row                                        # First row of charts
-------------------------------------
    
### Chart 1
Insert your r chunk for plot 1 here.

### Chart 2
Insert your r chunk for plot 2 here.  

Row {can play around row size here}        # Second row of charts
-------------------------------------
   
### Chart 3
Insert your r chunk for plot 3 here. 

### Chart 4
Insert your r chunk for plot 4 here. 

```

## Installing and loading packages 
You'll need to install the following packages in the console window using the `install.packages()` function. Depending on your dataset and visualisation goals you may require more [packages](https://rstudio.com/products/rpackages/) than those listed here.

```{R}
library(flexdashboard)         # Visualise related data as an interactive dashboard
library(tidyverse)             # Includes useful packages like ggplot2 (for plots) and dplyr (cleaning the data)
library(shinyWidgets)          # Customising Shiny applications         
library(ggpubr)                # Useful for researchers looking to easily compare means and plot the subsequent results 
library(shiny)                 # Creating interactive apps
```
## Loading the datatset
Once you've uploaded your dataset into your new project, you'll want to load it so that you can start working on the data. Give your dataset a name (eg `PD_original`) and then use a function relevant to the type of file you are trying to load (eg `read_csv` for a `.csv` file). 

```{R}
PD_original <- read_csv(file = "proj_data.csv")
```
## Cleaning the data
You want to call the changed dataset something new (eg `final_data`) so that you can always revisit the original, unfiltered dataset within your `R` project. The `dplyr` package within `tidyverse` allows you to filter the data as required for your analysis and visualisation goals. 

```{R}
final_data <- PD_original %>%                                    
  rename(                                      # Renaming some column variables for clarity
    Animals = Condition_Animals,                
    Control = Condition_Control,
    Luxury = Condition_Luxury,
    Memes = Condition_Memes,
    Positive_Affect = PositiveAffect,
    Negative_Affect = NegativeAffect,
    Life_Satisfaction = LifeSatisfaction5
    ) %>%
  filter(!is.na(SWB)) %>%                      # Removing NA values for the dataset
  select(-Finished, -Attrite) %>%.             # Remove redundant columns eg who did and didn't finish the study
  mutate(                                      # Adding levels (different experimental groups) to the 'Condition' variable which will be useful for later plots as well as ordering these levels so that they aren't in the default alphabetical order
    Condition = factor(                       
      Condition, 
      levels = c(
        "Control",
        "Animals",
        "Luxury",
        "Memes")
      ))
```
*Note*: The final score calculated in this dataset was the 'Subjective Wellbeing (SWB)' score since it was an average of the scores of the measures participants completed in the study. Therefore, any participants in the study that did not have a score in this column were considered an NA value. When the data was filtered this way, the remaining participant rows matched the researcher's total number of participants used in the analysis of the data. 

**For you** it might just be useful to filter rows where there is any NA value in the dataset using `filter(!is.na)`. Also another way to order your variables without mutate is to use the `scale_x_discrete()` function. 

## Preparing the data 
In this section you want to make your data reactive. This means that the output shown on the dashboard depends on some other expression that users can manipulate. For example, in this dataset the time spent looking at social media posts can be changed by users and the social media post conditions that are displayed in each Boxplot can be changed. Whatever is displayed within the Boxplot depends on the selections the user has made for these two hence making the data reactive.

```
Reactive <- reactive({                      # Makes the output dependent on some user input within the dashboard
  FD2 <- final_data %>%                     # FD2 will be plotted depending on the criteria in the following lines of code   
         filter(                          
           Condition == "Control" |         # Put | not , to ensure that the 'Control' box cannot be deselected since all comparisons are made with respect to the 'Control' condition 
           Condition %in% input$cond,       # Input from the Condition button selection (more on this below)
           Time_Images > input$time[1],     # Input from the Time slider selection (more on this below)
           Time_Images < input$time[2]) 
  })

TT <- final_data %>%                        # Second page of the flexdashboard is static as it is a snippet of a possible t-tests hence a separate dataset is used to the one above
  filter(Time_Images >= 5).                 # For the t-tests any rows were removed where the average time spent looking at a social media condition was less than 5 seconds

```

## Creating your own functions
This section is relevant when you have *almost* repetitive code. For instance, the first page of this dashboard is a 'Descriptive Statistics' page with 4 graphs (each for a measure of wellbeing). The graphs are all the same except for the y axis input (i.e. the averages for each measure of wellbeing). You could type the relevant code out for each graph in it's respective `server` chunk in `R` (more on this later) however this gets lengthy and neater code is always better. 

Creating a function (as shown below) allows you to 'automate' repetitive code and still choose variables that you'd like to change between graphs, for example the y axis. 

### Function for the descriptive statistics page

```{R}
pal <- c("#F8EE5C","#F9CA74","#FFA500","#FC8B03")      # Vector of colours for the boxplot fill

BoxPlot <- function(y_axis){                           # y axis input is the only thing to manually change between graphs
    ggplot(
      data = Reactive(),                               # Data is the Reactive() function created in the previous chunk
      mapping = aes_string(                            # Difference between aes() and aes_string() is explained below
        x = "Condition",                               # x axis is the 'Condition' variable with 4 levels 
        y = y_axis,                                    # This will be different for every graph, example code shown below
        fill = "Condition"                             # Changing the fill colour of each box in the boxplot
        )) +
    geom_boxplot(
      na.rm = TRUE                                     # Removing (redundant) missing value warning that appears because some 'Conditions' don't have data for time ranges selected by the slider
      ) +                                            
    theme_minimal() +                                  # Changing the theme to something more plain                        
    scale_fill_manual(                              
      values = pal,                                    # Manually filling in the colour for the boxplot using the vector created above
      guide = "none") +                                # Removing the legend that appears because of the use of fill
    labs(
      x = NULL,                                        # x axis label 'Condition' is redundant so remove 
      y = "Score") +                                   # Label the y axis as 'score' ie for each calculated wellbeing score
    theme(
      axis.text = element_text(face = "bold"),         # Bold the scales/ categories of each axis
      axis.title = element_text(face = "bold")) +      # Bold labels of each axis (in this case only y axis)
    ylim(1,5)                                          # Set the limit of the y axis scale from 1 to 5 so that you can observe
}                                                      # the maximum y value (5) on each graph
```
*Note*: When you are using ggplot in a function it is helpful to use `aes_string` not `aes` when mapping your aesthetics. This is because the variables loaded in `aes()` are searched for in the `global environment`. However, the function that you created does not get saved to the global environment because it is reactive (things saved in the `environment` are static ie they don't change depending on input). This means that the variables you're looking for won't be found. Instead `aes_string` goes to the reactive function and finds the variables you're looking for. 

### Function for the inferential statistics page

```{R}
BoxPlot_inf <- function(y_axis){                       # Again, y axis input is the only manual change possible between graphs
    ggplot(
      data = TT,                                       # Dataset used for t-test analyses
      mapping = aes_string(          
        x = "Condition",                               # x axis is the same as the descriptives page
        y = y_axis                                     # y axis differs for each plot
        )) +
    geom_boxplot( 
      fill = pal,                                      # Changing the fill colour of each box in the boxplot
      na.rm = TRUE) +                                  # Removing the missing value warning again
    theme_minimal() +                                  # Introducing a plain theme
    labs(
      x = NULL,                                        # Removing redundant x axis label
      y = "Score") +                                   # Changing y axis label to 'Score'
    theme(
      axis.text = element_text(face = "bold"),         # Making the axis scale/ category text bold
      axis.title = element_text(face = "bold")) +      # Making the axis label bold
    stat_compare_means(                                # This functions allows you to compare means with a reference group and add the p-values of these comparisons to a ggplot (part of the ggpubr package)
      label = "p.signif",                              # The way you'd like show your significance levels eg with an asterisk 
      method = "t.test",                               # Specify how you want the means to be compared ie through a t-test 
      ref.group = "Control",                           # Specify your reference group ie the group that you want other experimental conditions to be compared to
      label.y = 5.2,                                   # Elevating the significance level symbol slightly higher than boxes in the boxplot so they are clearly visible
      na.rm = TRUE)                                    # Removing redundant warnings
}
```        
### Some differences between the descriptive and inferential pages

You may have noticed that the way you fill the boxes between the descriptives and inferential code is different. This is because the inferential page has static data. So when you define your vector of 4 colours and code for a boxplot that has 4 boxes to be filled you can simply include the fill argument (using the vector) within the boxplot function. 
However if you try and do this for the descriptives page it only works for instances where there are 4 values (i.e. 4 boxes to be filled). Since the slider allows you to change the time ranges and not all of the conditions have data for every single range of time there will be instances less than 4 boxes will be plotted. This confuses a vector that has 4 named colours and an error appears `Error: Aesthetics must be either length 1 or the same as the data (4): fill`. One way to overcome this is to make the `fill` argument based on 'Conditions' (ie as an aesthetic within the ggplot) and just remove the legend that appears since it is redundant information.

Also, the descriptives page used the `ylim` function to set the y axis limits whereas the code for the inferential page did not include this function. This is because the inferential page makes use of the `label.y` argument to elevate the significance level symbols. Since the symbols now form the highest point of the plot the value 5 conicidentally appears on the y axis for individuals to observe. Hence including the `ylim` function in the code for the inferential page is redundant. 

### T-tests 

Another thing to note is that you can choose to run your t-tests and then plot your graphs separately. Using the [`stat_compare_means`](https://www.rdocumentation.org/packages/ggpubr/versions/0.2.5/topics/stat_compare_means) function is just a shorter way of doing the same thing. 

To run your t-test use the [`t.test()`](https://docs.google.com/document/d/1QjuqDvjvf9JetBTz399xiHdsYruCYJ_j6U0x1VHcLOo/edit) function. An example is given below for comparing the 'Control' condition to the 'Animals' condition on differences in positive affect. The output from your t-test will appear in the console window.

```{R}
# Firstly you need to create two data frames for the t-test 
  # One data frame is for the 'Animals' group
  
  anml <- final_data %>%
    filter(Condition == "Animals") %>%.                      # You want to only keep the 'Animal' rows in the 'Condition' column     
    select(Positive_Affect)                                  # You want the values for these rows from the 'Positive_Affect' column
    
  # The other data frame is for the 'Control' group
  
  ctrl <- final_data %>% 
    filter(Condition == "Control") %>%                       # You want to only keep the 'Control' rows in the 'Condition' column    
    select(Positive_Affect)                                  # You want the values for these rows from the 'Positive_Affect' column
    
t.test(anml, ctrl)                                           # Run your t-test
```
## Time for a break ...
![](https://media.giphy.com/media/xUNd9VbEByZfcm994c/giphy.gif)

## Creating the buttons and slider 

Creating sliders and buttons (shinyWidgets) is useful if you have variables that you'd like to change within the graph to observe their effects like time, gender or age. There are a number of options to choose from when selecting a [shinyWidget](https://shiny.rstudio.com/gallery/widget-gallery.html).

For this dataset some [buttons](https://shiny.rstudio.com/reference/shiny/latest/checkboxGroupInput.html) were included to allow individuals to select which Conditions they'd like to see compared to the 'Control' group. A [slider](https://shiny.rstudio.com/reference/shiny/latest/sliderInput.html) was also created to visualise the difference between group means when time spent looking at a social media post changes.

```{R}
checkboxGroupButtons(
   inputId = "cond",                # Giving the buttons a name that you will call on later for graph input
   label = "Conditions",            # Giving the buttons a label on the dashboard 
   status = "lightblue",            # Changing the colour of the buttons using CSS (more on this later)
   choices = c(                     # The available buttons 
     "Luxury",
     "Animals",
     "Memes"),   
   selected = c(                    # Which buttons are selected as a default when you first go onto the dashboard
     "Luxury",
     "Animals",
     "Memes"),
   individual = TRUE,               # Select TRUE to separate the buttons from each other
   width = "300px"                  # Play around with the sizing of the buttons
)

sliderInput(
  inputId = "time",                 # Used to access the value for the slider                  
  label = "Time (secs)",            # Slider label seen on the dashboard
  min = 0,                          # Min value able to be selected ie the shortest time spent looking at social media posts 
  max = 1600,                       # Max value able to be selected ie the longest time spent looking at social media posts
  value = c(0, 1600),               # Initial value selected by the slider when viewing the dashboard
  ticks = FALSE,                    # Removing ticks to make the slider smooth
  width = "400px"                   # Playing around with the slider size 
  )
```

## Plotting the graphs on the descriptive statistics page
You need two r chunks for this section. A `render` chunk is what is the user interface ie it is what it shown on the dashboard. It takes the input from the `server` chunk and produces the output on the dashboard. The `server` chunk calculates what you'd like the dashboard to show ie it the *behind the scenes* code. 

```
Positive Affect                # Chart 1 in row 1
{r, context="render"}          # render chunk     
plotOutput("PA")               # plot PA, PA is defined in the chunk below
```
```
{r, context="server"}
output$PA <- renderPlot({      # We are choosing to render a plot hence renderPlot is used
  BoxPlot("Positive_Affect")   # Call the BoxPlot function you created earlier and change the y axis input to data from the positive affect column 

})
```
This process is repeated for as many plots as you'd like to have.
Negative Affect
```
{r, context="render"}
plotOutput("Neg")
```
```
{r, context="server"}
output$Neg <- renderPlot({
  BoxPlot("Negative_Affect").  # Change the y axis input to include data from the negative affect column
})
```
Life Satisfaction
```
{r, context="render"}
plotOutput("LS")
```
```
{r, context="server"}
output$LS <- renderPlot({
  BoxPlot("Life_Satisfaction") # Change the y axis input to include data from the life satisfaction column
})
```
Subjective Wellbeing
```
{r, context="render"}
plotOutput("SWB")
```
```
{r, context="server"}
output$SWB <- renderPlot({    
  BoxPlot("SWB")               # Change the y axis input to include data from the subjective wellbeing column
})
```

## Plotting the graphs on the inferential statistics page
This page is static ie there are no `shinyWidgets` that can be modified by a user. However, you can code in `render` and `server` chunks as you did with the page above. 

```
Positive Affect                 # Chart 1 row 1
{r, context="render"}
plotOutput("PA_stat")
```
```
{r, context="server"}
output$PA_stat <- renderPlot({
BoxPlot_inf("Positive_Affect")  # Call the function created earlier for the inferential page and change the y axis to the relevant column ie positive affect 
})
```
Negative Affect
```
{r, context="render"}
plotOutput("NA_stat")
```
```
{r, context="server"}
output$NA_stat <- renderPlot({
BoxPlot_inf("Negative_Affect")   # Change to the relevant y axis
})
```
Life Satisfaction
```
{r, context="render"}
plotOutput("LS_stat")
```
```{r, context="server"}
output$LS_stat <- renderPlot({
BoxPlot_inf("Life_Satisfaction") # Change to the relevant y axis 
})
```
Subjective Wellbeing
```
{r, context="render"}
plotOutput("SWB_stat")
```
```{r, context="server"}
output$SWB_stat <- renderPlot({
BoxPlot_inf("SWB")               # Change to the relevant y axis
})
```

## Using CSS to change colours 

There are certain parts of the dashboard that you may want to change the colour of. Usually this is just a matter of including an argument within a function in `R`. However, sometimes it is difficult to find the relevant argument or function or even to figure out how to get it to work. For example, [changing the slider colour]( https://rdrr.io/cran/shinyWidgets/man/setSliderColor.html) using the relevant `R` function didn't seem to work with teh code provided above. An easy way of combatting this is to include CSS in your `.Rmd` file. There are plenty of available resources on learning CSS but for a fast way of [changing colours](https://www.w3schools.com/howto/howto_css_custom_checkbox.asp) you can: 

1. Run your dashboard within the `R Markdown` file
2. Once loaded right click on the element of the dashboard you'd like to change and select 'Inspect'
3. Look at the styles tab and copy the code of the relevant element you want to change
4. Paste the code in your `.Rmd` file under the title/ initial set up section (don't forget to wrap this code with `<style>`)
5. Change the hex code already there to a colour you'd like

For example this code was found using the above steps is provided below: 

```
<style>                               # Start wrap                 
.navbar {                             # Changing the background colour of the title bar
  background-color: #5CA3F3;          
  border: none;                       # Removing the border but if you want to keep the border as is delete this line of code
}
</style>                              # End wrap, note that it is different to the start wrap!
```
You could alternatively select a [theme](https://rstudio.github.io/shinythemes/) for your dashboard if you don't want to customise your own colours.
 
# So ... does social media affect our wellbeing?
Now that you have your working `flexdashboard` :clap: you can look at your plots and make conclusions about the data. Looking at the inferential page of this dashboard we can conclude that when individuals spend 5 seconds or more looking at images of cute animals negative emotions are reduced. However, pictures of our fluffy friends have no influence on our positive emotions, life satisfaction or subjective wellbeing. Also, images of luxury items and memes have no effect on any of these measures. 

The researchers raise a number of reasons for why this could be but as with any psychological research replication and revisiting our studies is always key. 

Incase you feel your negative emotions elevated after all of this hard work here's a cute a puppy. 

<img src="https://i.redd.it/dltqk6q2w0oz.jpg" width="200" height="350" title="cute puppy">

# Extra comments 

## Don't be afraid of Google
There are so many resources online to help you achieve the dashboard of your dreams. `R studio` has some great [cheatsheets](https://rstudio.com/resources/cheatsheets/) ... otherwise Google your question and you'll be sure to figure out a solution. 


## Making the code simple
As you work on a project overtime your code may get a bit messy, redundant or just lengthy. It is always good practice to go back and revisit your code and figure out creative ways of making it appear more readable, simple and concise. This may include grouping repetitive code or even finding functions that allow you to do two steps in one like the `stat_compare_means` function does.

## Additions to your flexdashboard for future projects
 
 * Figure out a way to make t-tests reactive input 
 * Run additional statistical analyses to create different types of plots eg multiple regression 

# Credits :tada:
Thanks to the [Cynical Scyntist](https://osf.io/f3vzn/) for uploading their study and the results on [OSF](https://osf.io/n6v8j/). 

