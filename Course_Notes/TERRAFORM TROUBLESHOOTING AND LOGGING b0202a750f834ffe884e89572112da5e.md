# TERRAFORM TROUBLESHOOTING AND LOGGING

There are many potential types of issues that you can experience with Terraform Language, State Core, or Provider errors.

To find errors, its important to have a readable and consistent configuration.

Terraform CLI includes a sub-command to format the configuration file into an idiomatic or canonical way and another command to validate the syntax. We've already discussed `terraform fmt` and `terraform validate`

`terraform fmt` only parses the configuration files, looking for formatting errors.

That is why, to check for configuration errors, you should run `terraform validate` after formatting your configuration.

If there is a provider error or any other error that still persists, you can enable detailed logging.

Set "TF_LOG" to TRACE, DEBUG, INFO, WARN, or ERROR
Example : `export TF_LOG=DEBUG`

TRACE is the most verbose one.
Variables and command are in all-caps!

Saving output to a file:
Example: `export TF_LOG_PATH=terraform.log`

You can generate logs from the core application and the Terraform provider, separately.

To enable core logging, set the TF_LOG_CORE environment variable

To generate provider logs, use the TF_LOG_PROVIDER to the appropriate log level.