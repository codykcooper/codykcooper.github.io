# Intro to Shiny Modules
This post is the first in a series that will hopefully provide a start to finish tutorial on how to build and deploy a shiny app. This series of posts will show how to: 
1. Build a shiny app using shiny modules
2. Dockerize your app
3. Deploy the docker image using [ShinyProxy](https://www.shinyproxy.io/)

I'll assume some familiarity with R and shiny throughout this blog post. But, those just starting with shiny may still find these posts helpful. For a more detailed introduction to shiny see [this](https://shiny.rstudio.com/tutorial/).

Throughout this app I'll be using a data set I found on kaggle documenting weekly avocado sales across the United States. The data is outdated, but highlights dimensions I commonly see as a data scientst. In this data set, we see a mix of geographic groupings in a single column (e.g., state, city, region), total sales + sales broken out by different products or product groupings, and a time compontent tracking overall business performance. See think [link](https://www.kaggle.com/neuromusic/avocado-prices) for more details on the data.

As a note, the point of this and upcoming posts is not necessarily to create a compelling app. Most of this app was built at the same time I was exploring / becoming more familiar with the avocado dataset. As such, the resulting app may not be particulalry compelling or tell a powerful story. When developing real apps, it will be important to be familiar with the dataset you are working with, and what would be useful to understand about your dataset given the needs of your end user. 

Ok, preamble done. 

The app will start with a basic skeleton for the UI and Server functions stored in a single app.R file. Below is the code:

```r
ui <- dashboardPage(

    dashboardHeader(title = "ShinyAvacado"),

    dashboardSidebar(
        sidebarMenu(
            menuItem("Overview", tabName = "overview", 
            icon = icon("dashboard")),

            menuItem("Data Vizualization", tabName = "dataviz", 
            icon = icon("bar-chart-o")),

            menuItem("Explore Explore", tabName = "eda", 
            icon = icon("cog"))
        )
    ),

    dashboardBody(
        tabItems(
            tabItem(tabName = "overview",
                    h2("Overview tab content")
            ),
            tabItem(tabName = "dataviz",
                    h2("Data Vizualization tab content")
            ),
            tabItem(tabName = "eda",
                    h2("Explore Data tab content")
            )
        )
    )
)

# Define server logic required to draw a histogram
server <- function(input, output) {
    # Server Functions
}

# Run the application
shinyApp(ui = ui, server = server)

```

I'm calling this app "ShinyAvocado" to really drive home what this app is, as silly shiny application showing some sales data for avocados.  The app will have 3 tabs displayed in the sidebar:
1. Overview - We will show quick summary tables of the data. 
2. Data Visualization - This will show a simple line chart of the time series with a selectable variable, region, and avocado type (convetional or organic). 
3. Data Exploration - This final tab will allow the user to create scatter plots for up to two variables from the data set. 

Each of these will utilize a shiny module stored in 4 different files. All files are available on the [github repo](https://github.com/codykcooper/shiny-module) that accompanies this post. The modules I will show in this post include:
1. [avo_filters.R](https://github.com/codykcooper/shiny-module/blob/master/R/avo_filter.R) - filter the time range of the data passed to the app.
2. [avo_overview.R](https://github.com/codykcooper/shiny-module/blob/master/R/avo_overview.R) - show a summary table of the data passed to the module.
3. [avo_linechart.R](https://github.com/codykcooper/shiny-module/blob/master/R/avo_linechart.R) -  a line chart of the line series data.  
4. [avo_explore.R](https://github.com/codykcooper/shiny-module/blob/master/R/avo_explore.R) - provide an interactive scatter plot where the user can plot up to two variables against eachother. 

## What is a shiny module?
All right. Now that the initial overview is done we'll start writing our first shiny module. A shiny module consists of paired r functions. There is an R function to generate the module UI, and a seperate R module to provide the server functionality. In other words, 1 function outline what the module looks like, and the other is the engine for getting stuff done. As recommended practice, it is best to append "UI" to the end of the function generating the UI. As an example, below is the code in `avo_filters.R` which contains two functions, `avo_filterUI` and `av_filter`. Lets start with `avo_filterUI`. 

The UI for avo_filter is very simple. All we are doing with this UI is subsetting the available data to be within a certain date range. So all we really need is to create a `dateRangeInput` object using the built in shiny module. However, in order to make this a shiny module there are two additional sreps we must take. 

### Namespace
Shiny module require you to provide a unique namespace (I won't provide too much detail on what this is, but see [here](https://shiny.rstudio.com/articles/modules.html) for more detail). For our purpose (i.e., oversimplified and ignoring some useful definitions), a namespace is a unique identifier that is associated with each call to the UI function (this will make more sense below). The `ns` function created by `NS(id)` will be append to the beginning of each input id we create in thie function. In order to make sure you understand what is happening, I recommend trying out using `NS()` in an R console. For example, try running the code:

```r
id = 'test'
ns <- NS(id)
ns('new')
```
### tagList
The object returned by a shiny module UI function is an r shiny `tagList`. Within the tagList you can place whatever kind of shiny UI you want. It can be nested columns, rows, input types, etc. that you will need in order to create the desired UI. 

Putting this together, the below R code is all we need to create the UI component of our first R module. 

```r
#' filter_controlsUI
#' @export
avo_filterUI <- function(id){
  ns <- NS(id)

  tagList(
    fluidRow(
      dateRangeInput(
        inputId = ns("date"),
        label = "Select Date Range"
        )
    )
  )
}
```

Now that the UI is defined, we will create the `avo_filter` function to create our desired server functionality. This server component of our module will look exactly like code you may have normally placed in the `server` spot of your app.R file. All shiny module server functions **MUST** start  start with the parameters *input*, *output*, and *session* **in that order**. You can pass any subsequent information after defining these three parameters. In our case, the only subsequent information needed is our `AppInfo` object defined at the beginning of the server function. 

The `avo_filter` function does 3 things:
1. update the dateRangeinput in `avo_filterUI` with the min and max dates from the SQL table. 
2. Queries the avocado SQL table for dates with the specified range and returns a data.frame of all data.
3. Assignes the returned data.frame to AppInfo$df

```r
#' avo_filter
#' @export
avo_filter <- function(input, output, session, AppInfo){

  observe({
    req(AppInfo$con)

    date_range = tbl(AppInfo$con, AppInfo$tbl_name) %>%
      summarise(
        date_min = min(Date),
        date_max = max(Date)
      ) %>% collect()

    updateDateRangeInput(
      session = session,
      inputId = "date",
      label = "Select Date Range",
      min = date_range$date_min,
      max = date_range$date_max,
      end = date_range$date_max,
      start = as.Date(date_range$date_max) - 61
    )
  })

  filter_data <- reactive({
    req(input$date)
    dmin <- input$date[1]
    dmax <- input$date[2]

    df_filtered = tbl(AppInfo$con, AppInfo$tbl_name) %>%
      filter(between(Date, dmin, dmax)) %>%
      collect() %>%
      mutate(Date = as.Date(Date)) %>%
      dplyr::arrange(Date)

    return(df_filtered)
  })

  observe({
    AppInfo$df <- filter_data()
  })
}
```
Now that the firt module is created, we can add it to our main `app.R` file. Since this module will be affecting all subsequent operations, I think it is best to place the UI component of the module in the main sidebar of the app. Below, I add the `avo_filterUI` function to the sidebar, and assign its functionality in the `server` function of the app using `callModule`. 

```r
library(ShinyAvacado)
library(shinydashboard)
# Define UI for application that draws a histogram
ui <- dashboardPage(
    dashboardHeader(title = "ShinyAvacado"),

    dashboardSidebar(
        sidebarMenu(
            avo_filterUI('filter_controls'),

            menuItem("Overview", tabName = "overview", 
            icon = icon("dashboard")),

            menuItem("Data Vizualization", tabName = "dataviz", 
            icon = icon("bar-chart-o")),

            menuItem("Explore Explore", tabName = "eda", 
            icon = icon("cog"))
        )
    ),

    dashboardBody(
        tabItems(
            tabItem(tabName = "overview",
                    h2("Overview tab content")
            ),
            tabItem(tabName = "dataviz",
                    h2("Data Vizualization tab content")
            ),
            tabItem(tabName = "eda",
                    h2("Explore Data tab content")
            )
        )
    )
)

# Define server logic required to draw a histogram
server <- function(input, output) {

    con = dbConnect(RSQLite::SQLite(), "Data/avocado.sqlite")

    AppInfo <- reactiveValues(
        df = NULL,
        con = con,
        session_info = list()
    )

    callModule(avo_filter, "filter_controls", AppInfo = AppInfo)

    output$avo_table <- DT::renderDataTable({
        AppInfo$df %>%
        DT::datatable()
    })

    onStop(function(){
        observe({
            dbDisconnect(AppInfo$con)
        })
    })
}

# Run the application
shinyApp(ui = ui, server = server)

```

