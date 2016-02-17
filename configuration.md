# Introduction #

Application configuration based on OTP environment variables

currently the following OTP environment variables can be configured:

| Variable    | Description                                   | Default |
|:------------|:----------------------------------------------|:--------|
| flv\_dir     | default location of video files for playback. | /tmp    |
| listen\_port | default port to start ErlMedia                | 1935    |

Varialbes can easily be overridden with a standard .config file when the .config file is given ont he comand line while starting Erlang.




At later stage more options will be added to define the location of video files (e.g. for live recording)