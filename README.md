# WIP: CodeScanning::Standard

'code-scanning-standard' is a gem to integrate Standard, also known as StandardRB, and GitHub's code scanning feature.

The repository is composed by two components:

1. The gem which can be installed in any Ruby application
2. A default GitHub action to jumpstart usage

The RubyGem adds a `SARIF` exporter to the Standard runner. GitHub's code scanning feature accepts a `SARIF` file with the 'results' (alerts) generated by the tool.

The action, is what will run Standard with the exporter.

> Note: you can only run the gem within your application, and have our own action that calls Standard.

See more in the Installation and Usage sections.

## Action Installation

The easiest way to install the integration, is this action template below. It will install the gem in your app and run it from you within GitHub's action environment.

To install the action, create a file: `.github/workflows/standard-analysis.yml`.

It should look like:

```yaml
# .github/workflows/standard-analysis.yml
name: "Standard"

on: [push]

jobs:
  standard:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
      - name: Set up Ruby 2.7.1
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 2.7.1
      # This step is not necessary if you add the gem to your Gemfile
      - name: Install Code Scanning integration
        run: bundle add code-scanning-standard --skip-install
      - name: Install dependencies
        run: bundle install
      - name: Standard run
        run: |
          bash -c "
            bundle exec standard --require code_scanning --format CodeScanning::SarifFormatter -o standard.sarif
            [[ $? -ne 2 ]]
          "

      - name: Upload Sarif output
        uses: github/codeql-action/upload-sarif@v1
        with:
          sarif_file: standard.sarif
```

## Gem installation & usage in a custom action

Note: this is not necessary if you use the action above.

To install the gem add this line to your application's Gemfile:

```ruby
gem 'code-scanning-standard'
```

Then, in your custom GitHub's action, you need to run Standard and make sure you give it the `SarifFormatter`:

```bash
bundle exec standardrb --require code_scanning --format CodeScanning::SarifFormatter -o standard.sarif
```

As a last step, make sure you upload the `standard.sarif` file to the code-scan integration. That will create the Code Scanning alerts.
Thus, add this step to your custom Standard workflow:

```yaml
- name: Upload Sarif output
  uses: github/codeql-action/upload-sarif@v1
  with:
    sarif_file: standard.sarif
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/andrewmcodes/code-scanning-standard. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [code of conduct](https://github.com/andrewmcodes/code-scanning-standard/blob/main/CODE_OF_CONDUCT.md).

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the Code::Scanning::Rubocop project's codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/andrewmcodes/code-scanning-standard/blob/main/CODE_OF_CONDUCT.md).
