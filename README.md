# Pullreq Analysis

An analysis and report of how pull requests work for Github

## Installation

Make sure that Ruby 1.9.3 is installed on your machine. You can 
try [RVM](https://rvm.io/), if it is not. Then, it should suffice
to do:

<pre>
gem install bundler
bundle install
gem install mysql2
</pre>

## Configuration 

The executable commands in this project inherit functionality from the
[GHTorrent](https://github.com/gousiosg/github-mirror) libraries. 
To work, they need the GHTorrent MongoDB data and a recent version of
the GHTorrent MySQL database. You can configure and run GHTorrent
using the guide on [this page](https://github.com/gousiosg/github-mirror/wiki/Setting-up-a-mirroring-cluster).

In addition to command specific arguments, the commands use the same
`config.yaml` file for specific connection details to external systems.  You
can find a template `config.yaml` file
[here](https://github.com/gousiosg/github-mirror/blob/master/config.yaml.tmpl).
The analysis scripts only are only interested in the connection details for
MySQL and MongoDB, and the location of a temporary directory 
(the `cache_dir` directory).

## Analyzing the data

The data analysis consists of two steps:

* Generating intermediate data files
* Analysing data files with R


####Generating intermediate files
To produce the required data files, first run the
`bin/pull_req_data_extraction.rb` script like so:

```bash
  ruby -Ibin bin/pull_req_data_extraction.rb -c config.yaml owner repo lang
```

where: 
* `owner` is the project owner
* `repo` is the name of the repository
* `lang` is the main repository language as reported by Github. At the moment,
only `ruby` and `java` are supported arguments.

Here are some examples of projects you may want to try:

```bash
rails rails ruby
puppetlabs puppet ruby
Katello katello ruby
rapid7 metasploit-framework ruby
spree spree ruby
gitlabhq gitlabhq ruby
diaspora diaspora ruby
mongoid mongoid ruby
fog fog ruby
resolve refinerycms ruby
enspiral loomio ruby
opscode chef ruby
mitchellh vagrant ruby
Shopify active_merchant ruby
sanger sequencescape ruby
JumpstartLab reading ruby
mojombo jekyll ruby
puppetlabs facter ruby
imathis octopress ruby
jbossas jboss-as java
hornetq hornetq java
infinispan infinispan java
neo4j community java
sakaiproject nakamura java
openmicroscopy openmicroscopy java
sonatype nexus java
elasticsearch elasticsearch java
SpringSource spring-integration java
cgeo c-geo-opensource java
dana-i2cat opennaas java
brooklyncentral brooklyn java
SpoutDev SpoutAPI java
ModeShape modeshape java
jenkinsci jenkins java
floodlight floodlight java
KentBeck junit java
netty netty java
SpringSource spring-framework java
intridea grape ruby
eventmachine eventmachine ruby
akka akka scala
scala scala scala
scala-ide scala-ide scala
harrah xsbt scala
scalaz scalaz scala
xbmc xbmc c
antirez redis c
mruby mruby c
libgit2 libgit2 c
zfsonlinux zfs c
ajaxorg cloud9 javascript
joyent node javascript
jquery jquery javascript
mrdoob three.js javascript
mbostock d3 javascript
```

The data extraction script extracts several variables
for each pull request and prints to `STDOUT` a comma-separated
line for each pull request using the following fields: 

* `pull_req_id`: The database id for the pull request
* `project_name`: The name of the project (same for all lines)
* `github_id`: The Github id for the pull request. Can be used to see the
actual pull request on Github using the following URL:
`https://github.com/#{owner}/#{repo}/pull/#{github_id}`
* `created_at`: The epoch timestamp of the creation date of the pull request
* `merged_at`: The epoch timestamp of the merge date of the pull request
* `closed_at`: The epoch timestamp of the closing date of the pull request
* `lifetime_minutes`: Number of minutes between the creation and the close of
the pull request
* `mergetime_minutes`: Number of minutes between the creation and the merge of
the pull request
* `team_size_at_merge`: The number of people that had committed to the
     repository directly (not through pull requests) in the period
     `(merged_at - 3 months, merged_at)`
* `num_commits`: Number of commits included in the pull request
* `num_comments`: Total number of comments (`num_commit_comments + num_issue_comments`)
* `files_changed`: Total number of files changed (added, remove, deleted) by the
pull request
* `perc_external_contribs`: % of commits commit from pull requests up to one month
before the start of this pull request
* `total_commits_last_month`: Number of commits
* `main_team_commits_last_month`: Number of commits to the repository during
the last month, excluding the commits coming from this and other pull requests
* `sloc`: Number of executable lines of code in the main project repo
* `src_churn`: Number of src code lines changed by the pull request
* `test_churn`: Number of test lines changed by the pull request
* `commits_on_files_touched`: Number of commits on the files touch by the
pull request during the last month
* `test_lines_per_1000_lines`: Number of test (executable) lines per 1000 executable lines
* `requester`: The developer that performed the pull request
* `prev_pullreqs`: Number of pull requests by developer up to the specific pull request
* `requester_succ_rate`: % of merged vs unmerged pull requests for developer

Lines reported are always executable lines (comments and whitespace have been
stripped out). To count test case related data, the script 
exploits the fact that Java and Ruby projects are organized using the
Maven or Gem conventions respectively. Test cases are recognized as
follows:

* Java: Files in directories under a `/test/` branch of the file tree are
considered test files. JUnit 4 test cases are recognized using the `@Test`
tag. For JUnit3, methods starting with `test` are considered as test methods.
Asserts are counted by "grepping" through the source code lines for `assert*`
statements.

* Ruby: Files under the `/test/` and `/spec/` directories are considered
test files. Test cases are recognized by "grepping" for `test*` (RUnit),
`should .* do` (Shoulda) and `it .* do` (RSpec) in the source file lines.

####Processing data with R


