# [Change] Change your blog URL 
baseurl = "https://liusven.github.io" 
# [Do not change] Language setting Chinese 
languageCode = "zh-CN" 
# [Do not change] If Chinese/Japanese/ Korean language. Please set to true in order for .Summary and .WordCount to execute correctly. 
hasCJKLanguage = true 
# [Change] Site title 
title = "强风" 
# [Do not change] Theme name 
theme = "Diaspora" 
# [Do not change] Theme directory 
themesDir = "./themes/" 
# [Do not change] Set the number of entries in the list page, the default 10 
paginate = 10 
# [Do not change] Disable the directory to lowercase 
disablePathToLower = true 
# [Do not change] Set the automatic summary (summary) characters The number is 50. If the article description is empty, the summary will automatically use the first 50 characters of the article as the summary. 
summaryLength = 50 

# [Custom] Set the icon of the list page to display the number of words in the default display true/false display / not display
[params.iconshow] 
    # display classification 
    category to true = 
    # is not displayed series (related) 
    Series to false = 
    # display tag 
    Tag to true = 

# [] does not change the Classification Tag Series category 
[Taxonomies] 
  category = "the Categories" 
  Tag = "Tags" 
  Series = "series" 
# [Do not change] Generate link configuration 
[permalinks] 
    post = "/:year/:month/:day/:slug" 
# [Change] How to change the personal parameters, this should be. 
[params] 
    # Tell me who you are 
    author = "Hopper" 
    bio = "Blogger - Programmer - Gopher" 
    location = "Earth" 
    site_description = "Hopper's hugo static blog"
    Copyright = "Powered by [Hugo](//gohugo.io). Theme by [PPOffice](http://github.com/ppoffice)." 
    avatar = "css/images/avatar.png" 
    # Enter your email address To display your Gravatar icon in the profile. If not set the theme 
    # will fallback to the avatar. 
    gravatar = "you@example.com" 
    logo = "css/images/logo.png" 
    disable_mathjax = false # set to true to disable MathJax 

    # define which types of pages should be shown. By default the type with the most regular pages 
    mainSections = ["post"] 

    # Format dates Go's time formatting 
    date_format = "January 2, 2006" 

    #Add custom assets with their paths relative to the static folder 
  custom_css = [] 
  custom_js = [] 

# [custom] navigation bar menu configuration 
[[params.menu]] 
    name = "About" 
    link = "/about/" 
    target = " _blank" 
[[params.menu]] 
    name = "Links" 
    link = "/links/" 
    target = "" 
[[params.menu]] 
    name = "Archives" 
    link = "/archives/" 
    target = "" 
[[ Params.menu]] 
    name = "Categories" 
    link = "/categories/" 
    target = "" 
[[params.menu]] 
    name = "Tags" 
    link = "/tags/" 
    target = "" 


[social] 
# TODO