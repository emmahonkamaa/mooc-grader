Course and exercise configuration
=================================

## Configuration files

Configuration is written as JSON or YAML inside subdirectories.
Each subdirectory holding an `index.json` or `index.yaml` is a
valid active course.

Dates will be parsed as '%Y-%m-%dT%H:%M:%SZ', '%Y-%m-%dT%H:%M:%S',
'%Y-%m-%d %H:%M:%S', '%Y-%m-%d %H:%M', '%Y-%m-%d %H' or '%Y-%m-%d'.
Durations are given in (int)(unit), where units are y, m, d, h or w.

1. ### course_key/index.[json|yaml]
	* The directory name acts as a course key, which is used in
		* URLs: `/course_key`
	* `name`: A public complete course name
	* `description` (optional): A private course description
	* `lang` (optional/a+): The default language.
	* `contact`: (optional/a+) A private contact email for course configuration
	* `contact_phone`: (optional) A private contact phone number for course responsible
	* `assistants`: (optional/a+) A list of assistant student ids
	* `start`: (optional/a+) The course instance start date
	* `end`: (optional/a+) The course instance end/archive date
	* `static_dir`: (optional) This subdirectory will be linked to URL /static/course_key
	* `exercises`: (deprecated, see modules) A list of active exercise keys
	* `modules`: a list of
		* `key`: part of the url
		* `name`,`title`: (optional/a+) The name of the course module
		* `status`: (optional/a+) ready/hidden/maintenance
		* `points_to_pass`: (optional/a+) limit to get passed marks
		* `introduction`: (optional/a+) introduction
		* `open`: (optional/a+) first access date
		* `close`: (optional/a+) deadline date
		* `duration`: (optional/a+) deadline in duration from open
		* `late_close`: (optional/a+) late deadline date
		* `late_duration`: (optional/a+) late deadline in duration from first deadline
		* `late_penalty`: (optional/a+) factor of points worth for late submission
		* `type`: (optional/a+) a key name in 'module_types'
		* `children`: a list of
			* `key`: part of the url
			* `config`: a path to exercise configuration OR
			* `static_content`: a path inside static directory
			* `category`: a key name in 'categories'
			* `name`,`title`: (optional/a+) The name of the learning object
			* `status`: (optional/a+) ready/unlisted/hidden/maintenance
			* `max_submissions`: (optional/a+)
			* `max_points`: (optional/a+)
			* `points_to_pass`: (optional/a+) limit to get passed marks
			* `min_group_size`: (optional/a+)
			* `max_group_size`: (optional/a+)
			* `allow_assistant_viewing`: (optional/a+) true or false
			* `allow_assistant_grading`: (optional/a+) true or false
			* `use_wide_column`: (optional/a+) true to loose third column
			* `generate_table_of_contents`: (optional/a+) show index of children
			* `type`: (optional/a+) a key name in 'exercise_types'
			* `children`: list recursion
	* `categories`: a list of
		* `name`: (optional/a+)
		* `status`: (optional/a+) ready/hidden
		* `points_to_pass`: (optional/a+) limit to get passed marks
	* `module_types`,`exercise_types`: keyed maps of default values
	* `numerate_ignoring_modules`: (optional/a+) true to numerate I:1...n, II:n+1...m

2. ### course_key/exercise_key.[json|yaml]
	* The file name acts as an exercise key, which is used in
		* URLs: `/course_key/exercise_key`
		* Must match the exercise list in `index.[json|yaml]`
	* `title`: A title of the exercise
	* `description` (optional): An exercise description (Dublin Core metadata)
	* `instructions` (optional): Most default templates will print given
		instructions HTML before the exercise widgets.
	* `max_points` (optional): The maximum exercise points (positive int).
		Overrides any maximum points reported by test actions.
	* `view_type`: A dotted name for an exercise implementation

	Rest of the attributes are exercise type specific.

3. ### course_key/sandbox
	A directory for sandbox environment content. Directory is copied into
	sandbox as `/usr/local/sandbox/course_key`. In a sandbox action the
	executables are automatically in the command path. Files beginning with
	`install-` will be run inside the sandbox in alphabetical order during the
	`manage_sandbox.sh` run. Before that the `scripts/sandbox` and the selected
	`scripts/sandbox_available` install files are run.

4. ### course_key/sandbox/from_sandbox_available
	List required files from `scripts/sandbox_available` each name at one line.

## Exercise view types

Common exercise views are implemented in `access.types` and they should fit most
purposes by configuration and templating. However, it is possible to implement a
course specific exercise view in a course specific Python module.

1. ### access.types.stdasync.acceptFiles
	Accepts named files for asynchronous grading queue. Extended attributes:
	* `files`: list of expected files
		* `field`: file field name
		* `name`: actual file name, may include subdirectories
	* `template` (default `access/accept_files_default.html`):
		name of a template to present
	* `accepted_message` (optional): overrides the default message displayed when
		asynchronous submission is accepted
	* `never_wait` (optional): true stops the automatic feedback polling for
		asynchronous submissions (normally occurs if queue is shorter than 3)
	* `feedback_template` (default `access/task_success.html`):
		name of a template used to format the feedback
	* `actions`: list of asynchronous test actions

2. ### access.types.stdasync.acceptPost
	Accepts form text for asynchronous grading queue. Extended attributes:
	* `fields`: list of text fields
		* `name`: field name and written file name
		* `title` (optional): field title or label
		* `more` (optional): more instructions
		* `required` (optional): `true` to require an answer
	* `template` (default `access/accept_post_default.html`):
		name of a template to present
	* `accepted_message` etc as in type 1.

3. ### access.types.stdasync.acceptGitAddress
	Writes the Git address into user/gitsource file for asynchronous grading
	queue. See grader.actions.git*. Extended attributes:
	* `require_gitlab` (optional):
		a host name for a Gitlab installation.
		Makes sure that the address is an SSH repo path or any HTTP URL
		in given Gitlab host. Stores the standard SSH path for key access.
	* `template` (default: `access/accept_git_default.html`):
		name of a template to present
	* `accepted_message` etc as in type 1.

4. ### access.types.stdasync.acceptAttachedExercise
	Accepts attached exercise rules and user files for asynchronous
	grading queue. Extended attributes:
	* `template` (default `access/accept_files_default.html`):
		name of a template to present.
		This test is used with exercises configured externally
		where also the creation of the form should be taken care off.
		Post should include several file fields named `content_N` where
		`content_0` is an exercise rule attachment and rest are
		user files which are named by `file_N` post fields.
		(Format is used by A+ exercises with attachments).
	* `accepted_message` etc as in type 1.

5. ### access.types.stdsync.createForm
	Synchronous form checker. Requires `max_points` in the
	exercise configuration. If form has no points configured then maximum
	points are granted on errorless submission. Extended attributes:
	* `fieldgroups`: list of field groups
		* `name` (optional): group name (fieldset legend)
		* `pick_randomly` (optional): number of fields to randomly sample
		* `group_errors` (optional): `true` to hide individual failed fields
		* `fields`: list of fields
			* `title` (optional): field title or label
			* `more` (optional): more instructions
			* `include` (optional): template name to include
				as content in more instructions
			* `type`: `radio`/`checkbox`/`dropdown`/`text`/`textarea`
			* `points` (optional): number of points to grant
			* `required` (optional): `true` to require an answer
			* `correct` (optional): exact correct answer for text fields
			* `regex` (optional): regex to match correct answer for text fields
			* `options` list of options for choice fields
				* `label`: option label
				* `correct` (optional): `true` for correct option.
					Checkbox requires all and only correct
					options selected. Radio requires one of
					the correct options selected. If no correct
					options are configured anything is correct.
	* `template` (default `access/create_form_default.html`): name of a template to present
	* `accepted_message` (optional): overrides the default message displayed when
		submission is accepted

6. ### access.types.stdsync.comparePostValues
	Synchronous check against posted values. Requires `max_points` in the
	exercise configuration. If values have no points configured then maximum
	points are granted on errorless submission. Extended attributes:
	* `values`: map of POST field names to rules:
		* `accept`: list of accepted values, [ False ] for no value, [ True ]
			for any value
		* `points` (optional): number of points to grant or negative to deduct
	* `template`: name of a template to present. Template should manually
		include a form that produces the expected POST values.

7. ### access.types.stdsync.noGrading
	Presents a template and does not grade anything. Extended attributes:
	* `template`: name of a template to present

## Test action types

Asynchronous exercises accept the submitted files into user/filename[s].
Once the queued grading commences the configured list of `actions`
will run in the listed order.

* Common attributes for each action are
	* `type`: A dotted name for a test action implementation
	* `points` (optional): Overrides points reported by the action.
		Passed action awards all points. Failed action awards zero points.
		Unless max points is separately set this will also set the max points
		for the action.
	* `max_points` (optional): Overrides max points reported by the action.
	* `title` (optional): grading action title for template
	* `html` (optional): true to pass output as HTML in template
	* `expect_success` (optional): true to not only stop but to set grading state
	 	to error when the action fails, "error" to write/mail error log

	Rest of the attributes are action type specific.

### Action types

Common test actions are implemented in `grader.actions` and they should
fit most purposes by configuration. However, it is possible to implement a
course specific test actions in a course_key.actions Python module. Typically
the actual tests should be written as programs or scripts run inside the safe
sandbox system.

1. ### grader.actions.prepare
	Does preparations on the submitted files. Additional attributes:
	* `attachment_pull` (optional): a TARGET file path to pull
		*user/exercise_attachment* from *acceptAttachedExercise* view
	* `attachment_unzip` (optional): `yes` to unzip *user/exercise_attachment*
		from *acceptAttachedExercise* into submission root
	* `unzip` (optional): a submitted user file name to unzip inside *user*
		directory
	* `charset` (optional): a character set where to convert submitted text
		files from any recognized character sets they are in
	* `cp_exercises` (optional): a space separated list of *path->path* where
		source path is relative to exercise configuration e.g.
		`exercise_dir->user` would copy contents of
		*exercises/course_key/exercise_dir* into the submission *user*
		directory.
	* `cp` (optional): a space separated list of *path->path* where both paths
		are relative to submission root e.g. `model/lib->user/lib` would
		replicate the lib directory among the user submitted files.
	* `mv` (optional): a space separated list of *path->path* where
		both paths are relative to submission root e.g.
		`user/file_name->user/new_dir/file_name`

	**Note** that the cp/mv *path->path* pattern does not replicate shell
	command arguments. Either dir->dir contents or individual file->file is
	inserted creating new parent directories if required.

2. ### grader.actions.sandbox
	Executes a command inside the chroot sandbox as a restricted user.
	Picks points from `TotalPoints: N` and/or `MaxPoints: N` lines if
	printed out. Additional attributes:
	* `cmd`: command line as an ARRAY
	* `dir` (optional): sandboxed path relative to submission root,
		default is *user*
	* `net` (optional): `true` to use network enabled sandbox user
	* `time` (optional): limit the seconds the command can run
	* `memory` (optional): limit the memory bytes (address space) the command
		can take, use 10k for kilobytes and 10m for megabytes
	* `files` (optional): limit the number of file descriptors the command can
		open
	* `disk` (optional): limit the disk bytes the command can write, use 10k
		for kilobytes and 10m for megabytes

3. ### grader.actions.sandbox_python_test
	Executes a command that should run a Python unittest inside the chroot
	sandbox as a restricted user. This is identical to the normal sandbox
	action except that the stderr (unittest output) is presented as stdout and
	the real stdout is nulled. Note that at the end of the complete test run
	(e.g. def tearDownClass) the `TotalPoints: N` and `MaxPoints: N` lines
	should be printed out.

4. ### grader.actions.gitlabquery
	Requires the *acceptGitAddress* view type with *require_gitlab* set.
	Queries the Gitlab API and checks desired properties. Additional attributes:
	* `token`: a Gitlab account private token for API access
	* `private` (optional): `yes` to stop if repository has public access
	* `forks` (optional): Project ID to stop if not forked from this project

5. ### grader.actions.gitclone
	Works with the *acceptGitAddress* view type. Tries to clone the
	repository. Additional attributes:
	* `files` (optional): a space separated list of files to select for
		submission from the repository. The files are moved into
		*user/filename[s]* and contents are listed in the feedback.
	* `repo_dir` (optional): the clone directory, default *user-repo*
	* `read` (optional): override the file where the git address is read from

6. ### grader.actions.expaca
	Executes the expaca testing application which compares outputs of a model
	and the student solutions. Expaca is not freely available. This action
	assumes that expaca is properly installed. Additional attributes:
	* `rule_file` (optional): default `checkingRule.xml`
	* `model_dir` (optional): default `model`
	* `user_dir` (optional): default `user`
	* `xslt_transform` (optional): a name of an XSL style file for
		transforming expaca XML output e.g. `expaca/xsl/aplus-utf8.xsl`

## Default sandbox scripts

Following common scripts are provided by default and copied into the sandbox.
However, the common scripts may have sandbox install requirements that are not
met by default. Both the /usr/local/sandbox and /usr/local/sandbox/course_key
are in the sandbox execution path and may be included in sandbox command line.

1. ### java_compile.sh
	Compiles all java files in submission. Takes arguments:
	* `--cp` (optional): classpath to use
	* `--clean` (optional): `yes` to remove java source after compilation

2. ### scala_compile.sh
	Compiles all scala files in submission. Takes arguments:
	* `--cp` (optional): classpath to use
	* `--clean` (optional): `yes` to remove scala source after compilation

3. ### virtualenv.sh envname cmd [arguments..]
	Activates the named Python virtualenv and passes rest for a command.

4. ### template.sh
	An example of a grading shell script. Will list submission files and
	always grade 10/10 points.

## Templates

Many type views can use a named template. The templates can be placed in
exercise directory and use subdirectories. The available variables are
listed below.

1. ### All templates
	* `course`: course configuration dictionary
	* `exercise`: exercise configuration dictionary

	Note that you can add any new keys to configuration and utilize them in templates.

2. ### Templates for asynchronous submissions
	* `result`: object holding POST results or None
		* `error`: True on failed POST
		* `missing_url`: True if no submission_url provided
		* `missing_files`: True if files missing
		* `missing_file_name`: True if file name missing
		* `invalid_address`: True if Gitlab address is rejected
		* `accepted`: True if accepted for grading
		* `wait`: True if the grading should be finished in a moment
	A default file submission form can be included with

		{% include 'access/accept_files_form.html' %}

3. ### Feedback templates for asynchronous submissions
	* `result`: object holding test results
		* `points`: total points granted
		* `max_points`: total maximum points
		* `tests`: entry for each test action
			* `points`: points granted
			* `max_points`: maximum points
			* `out`: test output
			* `err`: test errors
			* `stop`: True when rest of the actions
				were cancelled

4. ### Templates for createForm
	* `result`: object holding form and results
		* `form`: a Django form object
		* `accepted`: True if valid form POST was graded
		* `points`: granted points
		* `error_groups`: list of group_N names having errors
		* `error_fields`: list of field_N names having errors
	A default form can be included with

		{% include 'access/graded_form.html' %}

5. ### Templates for comparePostValues
	* `result`: object holding POST results or None
		* `accepted`: True
		* `received`: map of received POST fields => values
		* `points`: granted points
		* `failed`: list of failed field names