# frozen_string_literal: true

$LOAD_PATH.unshift(__dir__)

require "date"
require "slack-ruby-client"
require "reviewbot/github"

unless ENV["RACK_ENV"] == "production"
  require "dotenv"
  Dotenv.load
end

Slack.configure do |config|
  config.token = ENV["SLACK_API_TOKEN"]
  raise "Missing ENV[\"SLACK_API_TOKEN\"]" unless config.token
end

desc "Posts reviewable pull requests into a channel"
task :post_reviewable_pull_requests, %i[repositories labels channel] do |_, args|
  today = Date.today
  if today.saturday? || today.sunday?
    puts "Not posting on the weekend"
    next
  end

  slack_client = Slack::Web::Client.new
  github = ReviewBot::GitHub.new
  repositories = args.repositories.split("+")
  labels = args.labels.split("+")
  channel = args.channel

  pull_requests = github.reviewable_pull_requests(repositories: repositories, labels: labels)
  requested_reviewer = pull_requests[:requested_reviewer]
  need_review = pull_requests[:need_review]

  if requested_reviewer.empty? && need_review.empty?
    slack_client.chat_postMessage(channel: channel, text: "There are no pull requests that need to be reviewed.", as_user: true)
    next
  end

  unless requested_reviewer.empty?
    slack_client.chat_postMessage(channel: channel, text: "The following pull requests have requested a reviewer:", as_user: true)

    requested_reviewer.each do |hash|
      pull_request = hash[:pull_request]
      reviewers = hash[:reviewers]

      slack_client.chat_postMessage(
        channel: channel,
        as_user: true,
        attachments: [
          {
            fallback: "##{pull_request.number} - #{pull_request.title}:\n#{pull_request.html_url}\nReviewer: #{reviewers.join(", ")}",
            title: "##{pull_request.number}",
            text: "#{pull_request.title}:\n#{pull_request.html_url}\nReviewer: #{reviewers.join(", ")}",
            color: "#03b70b"
          }
        ]
      )
    end
  end

  unless need_review.empty?
    slack_client.chat_postMessage(channel: channel, text: "The following pull requests have no reviewer:", as_user: true)

    need_review.each do |pull_request|
      slack_client.chat_postMessage(
        channel: channel,
        as_user: true,
        attachments: [
          {
            fallback: "##{pull_request.number} - #{pull_request.title}:\n#{pull_request.html_url}",
            title: "##{pull_request.number}",
            text: "#{pull_request.title}:\n#{pull_request.html_url}",
            color: "#03b70b"
          }
        ]
      )
    end
  end
end
