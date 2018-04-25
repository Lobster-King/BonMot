require 'circleci_artifact'

# Make it more obvious that a PR is a work in progress and shouldn't be merged yet.
has_wip_label = github.pr_labels.any? { |label| label.include? "WIP" }
has_wip_title = github.pr_title.include? "[WIP]"

if has_wip_label || has_wip_title
    warn("PR is classed as Work in Progress")
end

# Warn when there is a big PR.
warn("Big PR") if git.lines_of_code > 500

# Mainly to encourage writing up some reasoning about the PR, rather than just leaving a title.
if github.pr_body.length < 3 && git.lines_of_code > 10
  warn("Please provide a summary in the Pull Request description")
end

src_root = File.expand_path('../', __FILE__)

SCHEME = "BonMot-iOS"

xcov.report(
  project: "#{src_root}/BonMot.xcodeproj",
  scheme: SCHEME,
  output_directory: "#{src_root}/build/#{SCHEME}/xcov",
  derived_data_path: "#{src_root}/build/derived_data"
)

## ** SWIFT LINT ***
# Use the SwiftLint included via CocoaPods
swiftlint.binary_path = "/usr/local/bin/swiftlint"
swiftlint.config_file = "#{src_root}/.swiftlint.yml"

# Run Swift-Lint and warn us if anything fails it
swiftlint.directory = src_root
swiftlint.lint_files inline_mode: true

# Getting artifact URLs from CircleCI

# You must set up the CIRCLE_API_TOKEN manually using these instructions
# https://github.com/Raizlabs/ios-template/tree/master/PRODUCTNAME#danger
token = ENV['CIRCLE_API_TOKEN']
# These are already in the Circle environment
# https://circleci.com/docs/2.0/env-vars/#build-specific-environment-variables
username = ENV['CIRCLE_PROJECT_USERNAME']
reponame = ENV['CIRCLE_PROJECT_REPONAME']
build = ENV['CIRCLE_BUILD_NUM']

if !(token.nil? or username.nil? or reponame.nil? or build.nil?)
  fetcher = CircleciArtifact::Fetcher.new(token: token, username: username, reponame: reponame, build: build)

  xcov = CircleciArtifact::Query.new(url_substring: 'xcov/index.html')
  slather = CircleciArtifact::Query.new(url_substring: 'slather/index.html')
  # screenshots = CircleciArtifact::Query.new(url_substring: 'screenshots/screenshots.html')
  xcpretty = CircleciArtifact::Query.new(url_substring: 'scan/report.html')
  xchtmlreport = CircleciArtifact::Query.new(url_substring: 'test_result/index.html')
  queries = [xcov, slather, xcpretty, xchtmlreport]
  results = fetcher.fetch_queries(queries)

  xcov_url = results.url_for_query(xcov)
  slather_url = results.url_for_query(slather)
  # screenshots_url = results.url_for_query(screenshots)
  xcpretty_url = results.url_for_query(xcpretty)
  xchtmlreport_url = results.url_for_query(xchtmlreport)

  if !xcpretty_url.nil?
    message "[Test Results: xcpretty](#{xcpretty_url})"
  else
    message "Tests in progress..."
  end

  if !xchtmlreport_url.nil?
    message "[Test Results: xchtmlreport](#{xchtmlreport_url})"
  else
    message "Tests in progress..."
  end

  if !xcov_url.nil?
    message "[Code Coverage: xcov](#{xcov_url})"
  end

  if !slather_url.nil?
    message "[Code Coverage: Slather](#{slather_url})"
  end

  # if !screenshots_url.nil?
  #   message "[Screenshots](#{screenshots_url})"
  # else
  #   message "Screenshots in progress..."
  # end
else
  warn "Missing CircleCI artifacts. Most likely the [CIRCLE_API_TOKEN](https://github.com/Raizlabs/circleci_artifact#getting-started) is not set, or Danger is not running on CircleCI."
end

# Test Reporting

junit.parse "#{src_root}/build/BonMot-iOS/scan/report.junit"
junit.report
junit.parse "#{src_root}/build/BonMot-OSX/scan/report.junit"
junit.report
junit.parse "#{src_root}/build/BonMot-tvOS/scan/report.junit"
junit.report