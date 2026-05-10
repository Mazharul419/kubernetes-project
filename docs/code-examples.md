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