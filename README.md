# taunt - taunts on command exit codes

#### USAGE
	taunt-bash [OPTIONS...] [ARGUMENT]

#### DESCRIPTION
	Read out messages to the user with espeak-ng based on the exit code of
	the previous command in bash.

#### OPTIONS
	-sym, --sym, -symbols, --symbols [FILE]
		List user defined symbols with exit codes. Load messages from 
		file if provided and list all names including signals, standard 
		exit codes and unnamed messages with their names and exit codes 
		after parsing, in addition to user defined symbols for which 
		messages are not displayed alongside.

	-sig, --sig, -signals, --signals, -signum, --signum
		List signals with exit codes that are read from system headers.

	-sys, --sys, -sysexit, --sysexit
		List standard exit codes names with exit codes that are read 
		from system header.

	-m, --m, -messages, --messages [FILE]
		List messages with exit codes. Load messages from file if 
		provided.

	-d, --d, -download, --download [URL]
		Pull messages from URL over HTTP. The host should provide 
		plaintext lines separated with linebreaks as response to a GET 
		request made using curl. Number of messages required equals 
		total of unassigned signals and standard exit codes and as many 
		lines are permuted at random from the HTTP response text. 
		Downloaded text is stored on disk and the list of messages with 
		exit codes is also stored separately.

	-r, --r, -reg, --reg, -register, --register [BASHRC]
		Register bash commands to toggle taunts feature in the shell. 
		Uses the .bashrc file provied, otherwise chooses the .bashrc 
		located in the user's home directory by default. On failure to 
		find the .bashrc there, the user's home directory is searched 
		for all files matching the name ".bashrc", prompting the user
		to choose the desired file. The original .bashrc is backed up in
		 the current directory. The script also searches for espeak-ng 
		on the system falling back to espeak, else reporting failure.

	-x, --x, -dereg, --dereg, -deregister, --deregister [BASHRC]
		Deregister bash commands to toggle taunts feature in the shell. 

	-c, --c, -clean, --clean
		Clear downloaded text and parsed messages.

	-l, --l, -locale, --locale [LOCALE_HINT]
		Set the locale to be used for espeak voice selection. System 
		locale is used by default.

	-h, --h, -help, --help
		Display usage information.

	[FILE]
		Load messages from file. See PARSING MESSAGES.

#### PARSING MESSAGES
	Messages which are formatted as "<name> <message>" where <name> matches
	 a parsed signal or standard exit code or user defined symbols symbol, 
	they are assigned to respective names. Otherwise the messages get 
	assigned to unassigned user defined symbols, followed by signals and 
	finally standard exit codes. Unused messages if any are assigned names of
	the form "MSG_XXXX" where "XXXX" denotes the size of the table 
	mapping exit codes to names and also the exit code for the message. After
	pulling messages from an URL and storing them on disk, the script only 
	parses them. User may provide a file containing messages to be parsed.

#### AUTOCOMPLETE
  taunt commands support argument autocompletion upon registraton. The
  script should be sourced in the shell to enable autocompleton.
