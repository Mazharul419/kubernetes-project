### Content Tabs

### Generic Content

=== "Plain text"

    This is some plain text

=== "Unordered list"

    * First item
    * Second item
    * Third item

=== "Ordered list"

    1. First item
    2. Second item
    3. Third item

=== "Code Blocks"

    An example code-block:

    ```tf title="ACM Certificate resource block" linenums="1" hl_lines="2-3"
    resource "aws_acm_certificate" "main" {
    domain_name       = "${var.subdomain}.${var.domain_name}"
    validation_method = "DNS"

    tags = {
        Name        = "${var.project_name}-${var.environment}-acm-cert"
    }
    }
    ```