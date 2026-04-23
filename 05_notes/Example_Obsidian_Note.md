# Header 1
## Header 2

### Header 3

*Italic*

**Bold**

- Bullet point list

1. Numbered List 

Code Blocks are started with three back ticks

```
This is a code block
# lines with comment sign will not run. These lines can be used to annotate code. 
```

Text in code blocks have no formatting and can be copied and pasted into command line. 

This is an example of what a code block could like in your work flow. 
```
# Denoising with dada2
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux_run2.qza \
  --p-trunc-len-f 150 \
  --p-trunc-len-r 150 \
  --p-n-threads 6 \
  --o-table table_run2.qza \
  --o-representative-sequences seqs_run2.qza \
  --o-denoising-stats dada2_stats_run2.qza
```

Some things to avoid
- Comments in the middle of the command
```
qiime dada2 denoise-paired \
  --i-demultiplexed-seqs demux_run2.qza \
  --p-trunc-len-f 150 \
  --p-trunc-len-r 150 \
  --p-n-threads 6 \
  --o-table table_run2.qza \
  --o-representative-sequences seqs_run2.qza \ # If you put a comment here the next line will not run
  --o-denoising-stats dada2_stats_run2.qza
```

-----------------------------------------------------------------------------------
Section Break (This is just a line of ------- across the whole page)

## Notes about viewing modes
- Obsidian has three viewing modes and that is edit, reading, and source mode.
- Edit and reading mode are turned off/on by the book/pencil icon in the top right hand corner. 
- If you see the book icon you are in edit mode and can edit your document. 
- If you see the pencil icon you are in reading mode and cannot make edits to your document. 
- If you press the 3 dots in the upper right hand corner you will see an option for source mode. If you select it a check will show up by the mode and obsidian will no longer automatically process the markdown language and you can see the symbols used for mark down formatting. For example you will see the hashtags before the headings. Where as when source mode is off obsidian automatically reads mark down format and makes into how it would look if it was rendered.. To deactivate this mode click the check and source mode will be turned off. This mode is similar to the code view on GitHub. 

------------------------------------------------------------------------
## Community plugins

 Obsidian has 100s of community plugins below is a list of some of the ones I use and what they do
- Highlightr - It gives you the ability highlight text
- Templater - It allows you to make templates out of obsidian notes
- Annotator - It allows you to open and annotate PDF and EPUB files
- Pandoc - It adds command palette options to export your notes to a variety of formats including Word Documents, PDFs, ePub books, HTML websites, PowerPoints and LaTeX among (many) others
- And many others if you can think of it then someone has probably made a community plugin. for it. Feel free to explorer and see what plugins sound interesting/helpful to you. 