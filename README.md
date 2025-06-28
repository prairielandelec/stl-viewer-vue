>[!WARNING]
>If you use any binary files in a git repo (like an STL) make sure you modify your repo's .gitattributes file (you may need to create it but many init scripts for web frameworks like Vue will do this automatically for common binaries)
>
>`\project_root\.gitattributes`
>```gitattributes
>* text eol=lf
>*.txt text eol=crlf
>
>*.stl binary
>```
>More info can be found on my [Blog Post on the subject](https://www.goofin.dev/article/STLsInGitRepos/)

# stl-viewer-vue
I found the great [stl-viewer](https://github.com/ottobonn/stl-viewer) project from Travis Geis but wanted to use it within a vuepress website I am working on.

I could have used it as is, but I wanted to use the npm version of the three js library to take advantage of the GPU accelerated rendering that is now mainline and also use the vue lifecycle hooks to handle resource management.

This is mostly the same logic but allows for a few passed in props to modify rotation and zoom as it is sometimes not quite right by default.

An example of use would be:

`<StlViewer style="height:300px;" model="/3d_models/3DBenchy.stl"  :flipX="true" :zoomIn=1.5 />`

Eventually I will come back to this and make an OBJ version as I would like to be able to show full assemblies as individual components with their own materials.
