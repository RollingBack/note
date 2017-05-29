=====================
编程工具
=====================

----------------------------------------------------------------
Sublime Text
----------------------------------------------------------------

工欲善其事，必先利其器。《论语》

**1.** 安装后如何设置？

把Preference下的Setting-Default的内容复制覆盖掉Setting-User的内容，然后修改这个文件。

.. code-block:: javascript
	:linenos:

	// While you can edit this file, it's best to put your changes in
	// "User/Preferences.sublime-settings", which overrides the settings in here.
	//
	// Settings may also be placed in file type specific options files, for
	// example, in Packages/Python/Python.sublime-settings for python files.
	{
	    // Sets the colors used within the text area
	    "color_scheme": "Packages/Color Scheme - Default/Monokai.tmTheme",

	    // Note that the font_face and font_size are overriden in the platform
	    // specific settings file, for example, "Preferences (Linux).sublime-settings".
	    // Because of this, setting them here will have no effect: you must set them
	    // in your User File Preferences.
	    "font_face": "Consolas",
	    "font_size": 14,

	    // Valid options are "no_bold", "no_italic", "no_antialias", "gray_antialias",
	    // "subpixel_antialias", "no_round" (OS X only) and "directwrite" (Windows only)
	    "font_options": [],

	    // Characters that are considered to separate words
	    "word_separators": "./\\()\"'-:,.;<>~!@#$%^&*|+=[]{}`~?",

	    // Set to false to prevent line numbers being drawn in the gutter
	    "line_numbers": true,

	    // Set to false to hide the gutter altogether
	    "gutter": true,

	    // Spacing between the gutter and the text
	    "margin": 4,

	    // Fold buttons are the triangles shown in the gutter to fold regions of text
	    "fold_buttons": true,

	    // Hides the fold buttons unless the mouse is over the gutter
	    "fade_fold_buttons": true,

	    // Columns in which to display vertical rulers
	    "rulers": [],

	    // Set to true to turn spell checking on by default
	    "spell_check": false,

	    // The number of spaces a tab is considered equal to
	    "tab_size": 4,

	    // Set to true to insert spaces when tab is pressed
	    "translate_tabs_to_spaces": false,

	    // If translate_tabs_to_spaces is true, use_tab_stops will make tab and
	    // backspace insert/delete up to the next tabstop
	    "use_tab_stops": true,

	    // Set to false to disable detection of tabs vs. spaces on load
	    "detect_indentation": true,

	    // Calculates indentation automatically when pressing enter
	    "auto_indent": true,

	    // Makes auto indent a little smarter, e.g., by indenting the next line
	    // after an if statement in C. Requires auto_indent to be enabled.
	    "smart_indent": true,

	    // Adds whitespace up to the first open bracket when indenting. Requires
	    // auto_indent to be enabled.
	    "indent_to_bracket": false,

	    // Trims white space added by auto_indent when moving the caret off the
	    // line.
	    "trim_automatic_white_space": true,

	    // Disables horizontal scrolling if enabled.
	    // May be set to true, false, or "auto", where it will be disabled for
	    // source code, and otherwise enabled.
	    "word_wrap": "auto",

	    // Set to a value other than 0 to force wrapping at that column rather than the
	    // window width
	    "wrap_width": 0,

	    // Set to false to prevent word wrapped lines from being indented to the same
	    // level
	    "indent_subsequent_lines": true,

	    // Draws text centered in the window rather than left aligned
	    "draw_centered": false,

	    // Controls auto pairing of quotes, brackets etc
	    "auto_match_enabled": true,

	    // Word list to use for spell checking
	    "dictionary": "Packages/Language - English/en_US.dic",

	    // Set to true to draw a border around the visible rectangle on the minimap.
	    // The color of the border will be determined by the "minimapBorder" key in
	    // the color scheme
	    "draw_minimap_border": false,

	    // If enabled, will highlight any line with a caret
	    "highlight_line": false,

	    // Valid values are "smooth", "phase", "blink", "wide" and "solid".
	    "caret_style": "smooth",

	    // Set to false to disable underlining the brackets surrounding the caret
	    "match_brackets": true,

	    // Set to false if you'd rather only highlight the brackets when the caret is
	    // next to one
	    "match_brackets_content": true,

	    // Set to false to not highlight square brackets. This only takes effect if
	    // match_brackets is true
	    "match_brackets_square": true,

	    // Set to false to not highlight curly brackets. This only takes effect if
	    // match_brackets is true
	    "match_brackets_braces": true,

	    // Set to false to not highlight angle brackets. This only takes effect if
	    // match_brackets is true
	    "match_brackets_angle": false,

	    // Enable visualization of the matching tag in HTML and XML
	    "match_tags": true,

	    // Highlights other occurrences of the currently selected text
	    "match_selection": true,

	    // Additional spacing at the top of each line, in pixels
	    "line_padding_top": 0,

	    // Additional spacing at the bottom of each line, in pixels
	    "line_padding_bottom": 0,

	    // Set to false to disable scrolling past the end of the buffer.
	    // On OS X, this value is overridden in the platform specific settings, so
	    // you'll need to place this line in your user settings to override it.
	    "scroll_past_end": true,

	    // This controls what happens when pressing up or down when on the first
	    // or last line.
	    // On OS X, this value is overridden in the platform specific settings, so
	    // you'll need to place this line in your user settings to override it.
	    "move_to_limit_on_up_down": false,

	    // Set to "none" to turn off drawing white space, "selection" to draw only the
	    // white space within the selection, and "all" to draw all white space
	    "draw_white_space": "selection",

	    // Set to false to turn off the indentation guides.
	    // The color and width of the indent guides may be customized by editing
	    // the corresponding .tmTheme file, and specifying the colors "guide",
	    // "activeGuide" and "stackGuide"
	    "draw_indent_guides": true,

	    // Controls how the indent guides are drawn, valid options are
	    // "draw_normal" and "draw_active". draw_active will draw the indent
	    // guides containing the caret in a different color.
	    "indent_guide_options": ["draw_normal"],

	    // Set to true to removing trailing white space on save
	    "trim_trailing_white_space_on_save": false,

	    // Set to true to ensure the last line of the file ends in a newline
	    // character when saving
	    "ensure_newline_at_eof_on_save": false,

	    // Set to true to automatically save files when switching to a different file
	    // or application
	    "save_on_focus_lost": false,

	    // The encoding to use when the encoding can't be determined automatically.
	    // ASCII, UTF-8 and UTF-16 encodings will be automatically detected.
	    "fallback_encoding": "Western (Windows 1252)",

	    // Encoding used when saving new files, and files opened with an undefined
	    // encoding (e.g., plain ascii files). If a file is opened with a specific
	    // encoding (either detected or given explicitly), this setting will be
	    // ignored, and the file will be saved with the encoding it was opened
	    // with.
	    "default_encoding": "UTF-8",

	    // Files containing null bytes are opened as hexadecimal by default
	    "enable_hexadecimal_encoding": true,

	    // Determines what character(s) are used to terminate each line in new files.
	    // Valid values are 'system' (whatever the OS uses), 'windows' (CRLF) and
	    // 'unix' (LF only).
	    "default_line_ending": "system",

	    // When enabled, pressing tab will insert the best matching completion.
	    // When disabled, tab will only trigger snippets or insert a tab.
	    // Shift+tab can be used to insert an explicit tab when tab_completion is
	    // enabled.
	    "tab_completion": true,

	    // Enable auto complete to be triggered automatically when typing.
	    "auto_complete": true,

	    // The maximum file size where auto complete will be automatically triggered.
	    "auto_complete_size_limit": 4194304,

	    // The delay, in ms, before the auto complete window is shown after typing
	    "auto_complete_delay": 50,

	    // Controls what scopes auto complete will be triggered in
	    "auto_complete_selector": "source - comment",

	    // Additional situations to trigger auto complete
	    "auto_complete_triggers": [ {"selector": "text.html", "characters": "<"} ],

	    // By default, auto complete will commit the current completion on enter.
	    // This setting can be used to make it complete on tab instead.
	    // Completing on tab is generally a superior option, as it removes
	    // ambiguity between committing the completion and inserting a newline.
	    "auto_complete_commit_on_tab": false,

	    // Controls if auto complete is shown when snippet fields are active.
	    // Only relevant if auto_complete_commit_on_tab is true.
	    "auto_complete_with_fields": false,

	    // By default, shift+tab will only unindent if the selection spans
	    // multiple lines. When pressing shift+tab at other times, it'll insert a
	    // tab character - this allows tabs to be inserted when tab_completion is
	    // enabled. Set this to true to make shift+tab always unindent, instead of
	    // inserting tabs.
	    "shift_tab_unindent": false,

	    // If true, the copy and cut commands will operate on the current line
	    // when the selection is empty, rather than doing nothing.
	    "copy_with_empty_selection": true,

	    // If true, the selected text will be copied into the find panel when it's
	    // shown.
	    // On OS X, this value is overridden in the platform specific settings, so
	    // you'll need to place this line in your user settings to override it.
	    "find_selected_text": true,

	    // When drag_text is enabled, clicking on selected text will begin a
	    // drag-drop operation
	    "drag_text": true,

	    //
	    // User Interface Settings
	    //

	    // The theme controls the look of Sublime Text's UI (buttons, tabs, scroll bars, etc)
	    "theme": "Default.sublime-theme",

	    // Set to 0 to disable smooth scrolling. Set to a value between 0 and 1 to
	    // scroll slower, or set to larger than 1 to scroll faster
	    "scroll_speed": 1.0,

	    // Controls side bar animation when expanding or collapsing folders
	    "tree_animation_enabled": true,

	    // Makes tabs with modified files more visible
	    "highlight_modified_tabs": false,

	    "show_tab_close_buttons": true,

	    // Show folders in the side bar in bold
	    "bold_folder_labels": false,

	    // OS X 10.7 only: Set to true to disable Lion style full screen support.
	    // Sublime Text must be restarted for this to take effect.
	    "use_simple_full_screen": false,

	    // OS X only. Valid values are true, false, and "auto". Auto will enable
	    // the setting when running on a screen 2880 pixels or wider (i.e., a
	    // Retina display). When this setting is enabled, OpenGL is used to
	    // accelerate drawing. Sublime Text must be restarted for changes to take
	    // effect.
	    "gpu_window_buffer": "auto",

	    // Valid values are "system", "enabled" and "disabled"
	    "overlay_scroll_bars": "system",

	    //
	    // Application Behavior Settings
	    //

	    // Exiting the application with hot_exit enabled will cause it to close
	    // immediately without prompting. Unsaved modifications and open files will
	    // be preserved and restored when next starting.
	    //
	    // Closing a window with an associated project will also close the window
	    // without prompting, preserving unsaved changes in the workspace file
	    // alongside the project.
	    "hot_exit": true,

	    // remember_open_files makes the application start up with the last set of
	    // open files. Changing this to false will have no effect if hot_exit is
	    // true
	    "remember_open_files": true,

	    // OS X only: When files are opened from finder, or by dragging onto the
	    // dock icon, this controls if a new window is created or not.
	    "open_files_in_new_window": true,

	    // OS X only: This controls if an empty window is created at startup or not.
	    "create_window_at_startup": true,

	    // Set to true to close windows as soon as the last file is closed, unless
	    // there's a folder open within the window. This is always enabled on OS X,
	    // changing it here won't modify the behavior.
	    "close_windows_when_empty": false,

	    // Show the full path to files in the title bar.
	    // On OS X, this value is overridden in the platform specific settings, so
	    // you'll need to place this line in your user settings to override it.
	    "show_full_path": true,

	    // Shows the Build Results panel when building. If set to false, the Build
	    // Results can be shown via the Tools/Build Results menu.
	    "show_panel_on_build": true,

	    // Preview file contents when clicking on a file in the side bar. Double
	    // clicking or editing the preview will open the file and assign it a tab.
	    "preview_on_click": true,

	    // folder_exclude_patterns and file_exclude_patterns control which files
	    // are listed in folders on the side bar. These can also be set on a per-
	    // project basis.
	    "folder_exclude_patterns": [".svn", ".git", ".hg", "CVS"],
	    "file_exclude_patterns": ["*.pyc", "*.pyo", "*.exe", "*.dll", "*.obj","*.o", "*.a", "*.lib", "*.so", "*.dylib", "*.ncb", "*.sdf", "*.suo", "*.pdb", "*.idb", ".DS_Store", "*.class", "*.psd", "*.db"],
	    // These files will still show up in the side bar, but won't be included in
	    // Goto Anything or Find in Files
	    "binary_file_patterns": ["*.jpg", "*.jpeg", "*.png", "*.gif", "*.ttf", "*.tga", "*.dds", "*.ico", "*.eot", "*.pdf", "*.swf", "*.jar", "*.zip"],

	    // List any packages to ignore here. When removing entries from this list,
	    // a restart may be required if the package contains plugins.
	    "ignored_packages": ["Vintage"]
	}


----------------------------------------------------------------
PHPStorm
----------------------------------------------------------------

**1.** 一直感觉很卡怎么办？

最好的办法是给电脑装SSD并把系统和软件都放在SSD上，或者用编辑器。

----------------------------------------------------------------
NotePad++
----------------------------------------------------------------

**1.** 在哪里设置字体？

设置下语言格式设置，更换字体后把使用全局字体和使用全局字体大小后生效。


-------------------------------------------------------------
电脑
-------------------------------------------------------------

**1.** 电脑反应慢。

原因有很多，硬件上可能硬件资源本身不是很足，软件上，中毒或者系统垃圾太多，甚至夏天
到了，电脑太热都会卡。硬件上不用太关注CPU，近几年的主流CPU的性能都是很好的，对于普
通的编程或者办公来说，已经很好了。显卡如果不玩游戏，没必要追求高配置，你需要关注的
是内存和硬盘。如果你的电脑是OEM的笔记本或者台式机，那么极有可能电脑上只有一条内存，
通常是2G或者4G，你需要自己再买一条加上，以便组成双通道。还有就是为你的电脑加一块SSD
作为系统盘，会极大的提升你的电脑使用体验和工作效率。夏天到了，电脑发热问题需要重视，
本来天气热人就心情烦躁，电脑因为过热而使CPU开启过热保护，运行效率低下，电脑开始卡，
风扇开始像直升机一样在你耳边咆哮，你还淡定吗？自己动手能力强的可以自己搞定，不强的找
强的搞定，或者花钱搞定。软件上，现在中毒不是很大的问题，但养成良好的使用习惯，可以节
省你很多时间，同时不会让你系统变的臃肿和缓慢。不要有软件弹个框让你装，你就毫不犹豫的
装，现在的软件通常都会在系统服务里加服务，同时在后台占用系统资源，好在特定的情况下
弹框打广告，或者后台向云上交互数据。没有用的软件不要开启，彻底关闭。

---------------------------------------------------------------
命令行工具
---------------------------------------------------------------
**1.** oh my zsh是Mac OS 下的一个shell实现，使用方便。

`oh my zsh 官方网站 <http://ohmyz.sh/>`_

安装方法：

.. code-block:: sh

	$ curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh

**2.** htop是个终端模式下一个top替代方案，图形化的界面让人爱不释手。

安装方法：

.. code-block:: sh
	
	$ yum install htop       #centos
	$ apt get install htop   #ubuntu
	$ brew install htop      #mac os x

**3.** homebrew是个mac下的包管理器，类似于yum(centos)、apt(ubuntu)。

`homebrew 官方网站 <http://brew.sh/>`_

安装方法：

.. code-block:: sh

	ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

**4.** Cygwin是个Windows的linux终端仿真器。

`Cygwin 官方网站 <http://www.cygwin.com/>`_

**5.** iTerm2是Mac下的一个终端仿真器。适合终端的重度用户。

`iterm2官方网站 <http://www.iterm2.com/>`_