# The Framey Ruby Gem

This gem can be used for easy Rails integration with the Framey video recording service. See http://framey.com for more information.

# Dependencies:

* httparty (ruby gem)
* will_paginate (ruby gem)

# Installation

First, sign up at http://framey.com to get an API key and secret and to set your callback url. 

If not using bundler, do:

`gem install framey`

otherwise in your Gemfile:

`gem framey`

There are two options for using the framey gem: using included scaffold code for views, models and callbacks or creating your own views.  Depending on your choice, follow the appropriate instructions below to complete the installation. 

## Using the included scaffold	

Run the framey generator:

`rails generate framey API_KEY API_SECRET`

This automatically creates default views, controller and routes for recording and viewing videos, as well as the callback that framey.com pings with the video information:

	/framey/videos
	/framey/videos/new
	/framey/videos/<video_id>
	/framey/callback

Create the supporting database tables:

	rake db:migrate

Edit the default configuration options in config/framey.rb:

	API_HOST = "http://framey.com"
	RUN_ENV = "production"
	API_KEY = "API_KEY_VALUE"
	SECRET = "API_SECRET_VALUE"
	API_TIMEOUT = 15
	MAX_TIME = 30

## Creating your own views

Create a framey initializer at config/framey.rb and modify the following code to use your API keys
	
	module Framey
	  API_HOST = "http://framey.com"
	  RUN_ENV = "production"
	  API_KEY = "API_KEY_VALUE"
	  SECRET = "API_SECRET_VALUE"
	  API_TIMEOUT = 15
	  MAX_TIME = 30
	end
	


# User / Development Flow

* You make a page on your site that displays the Framey flash video recorder
* Your user visits that page and clicks record
* If your user likes the video she just recorded, she clicks "Publish"
* The user's video is processed and stored on the Framey servers.
* Framey emails you and/or pings your servers at a specified callback url with a url to the video file
* You can choose to store those urls on your end, or just the id of the Framey video to access it later on Framey via an API call
* You then display the video to your user using either the Framey video player or your own favorite flash or HTML 5 video player

# Usage


To render the Framey video recorder in an ActionView (example assumes ERB) do:

    <%= javascript_include_tag "swfobject" %>
    <%= render_recorder({
      :id => "myRecorder",             # id for the flash embed object (optional, random by default)
      :max_time => 60,                 # maximum allowed video length in seconds (optional, defaults to 30)
      :session_data => {               # custom parameters to be passed along to your app later (optional)
        :user_id => <%= @user.id %>    # you may, for example, want to relate this recording to a specific user in your system
      }
    }) %>
    
When your user then views this recorder, records a video, and clicks "Publish", your app receives a POST to the specified callback url with the following parameters:

    {
      :video => {
        :name => [video name],                      # this is the video's UUID within the Framey system
        :filesize => [filesize],                    # the filesize in bytes of the video
        :duration => [duration],                    # the duration of the video in seconds
        :state => [state],                          # the state of the video, in this case "uploaded"
        :views => [number of views],                # the number of times this video has been viewed, in this case 0
        :data => [the data hash you specified],     # this is the exact data you specified in the :session_data hash when rendering the recorder
        :flv_url => [video url],                    # url to the flash video file on framey that you can feed into a video player later
        :mp4_url => [h.264 video url],              # url to the h.264 video file on framey that you can feed into a video player later
        :large_thumbnail_url => [thumbnail url]     # url to the large thumbnail image that was generated for this video 
        :medium_thumbnail_url => [thumbnail url]    # url to the medium thumbnail image that was generated for this video 
        :small_thumbnail_url => [thumbnail url]     # url to the small thumbnail image that was generated for this video                 
      }
    } 
    
To render the Framey video player in an ActionView (example assumes ERB) do:

    <%= javascript_include_tag "swfobject" %>
    <%= render_player({
      :id => "myPlayer",                    # id for the flash embed object (optional, random by default)
      :video_url => "[video url]",          # the video url received in the callback (required)
      :thumbnail_url => "[thumbnail url]",  # the thumbnail url received in the callback (required)
      :progress_bar_color => "0x123456",    # the desired color for the progress bar (optional, defaults to black)
      :volume_bar_color => "0x123456",      # the desired color for the volume bar (optional, defaults to black)
    })%>
    
Note that you don't have to use the Framey video player, though, and can use any other flash video player you'd like.

To get updated stats information about a given video, do:

    @video = Framey::Api.get_video("[framey video name]")
    
This returns a simple Framey::Video object, like so:

    #<Framey::Video:0x1037b4450 @state="uploaded", @filesize=123456, @name="c96323e0-54b1-012e-9d34-7c6d628c53d4", @large_thumbnail_url="http://framey.com/thumbnails/c96323e0-54b1-012e-9d34-7c6d628c53d4.jpg", @medium_thumbnail_url="http://framey.com/thumbnails/c96323e0-54b1-012e-9d34-7c6d628c53d4.jpg", @small_thumbnail_url="http://framey.com/thumbnails/c96323e0-54b1-012e-9d34-7c6d628c53d4.jpg", @data={"user_id" => 1}, @flv_url="http://framey.com/videos/source/c96323e0-54b1-012e-9d34-7c6d628c53d4/source.flv", @mp4_url="http://framey.com/videos/source/c96323e0-54b1-012e-9d34-7c6d628c53d4/source.mp4", @views=12, @duration=15.62>
    
To delete a video on Framey, do:

    @video.delete!

or:

    Framey::Api.delete_video("[framey video name]")

# Other Documentation

* http://rubydoc.info/gems/framey/1.2.0/frames

# License

(The MIT License)

Copyright (c) 2011 FIX

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
