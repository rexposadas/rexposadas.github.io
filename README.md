# Blog page

## Trouble shooting

### yll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)

You are missing a dependency. fix: 

`bundle add webrick`