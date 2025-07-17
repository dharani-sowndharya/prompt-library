# Aurora Postgres Provisioned Database Design Rules

*Comprehensive rules for designing AWS Aurora PostgreSQL Provisioned databases using Terraform, following AWS Well-Architected Framework principles and industry best practices for production-ready, secure, and cost-effective database infrastructure.*

## Context

*These rules apply when designing and implementing Aurora PostgreSQL Provisioned clusters using Infrastructure as Code (Terraform) for production environments.*

**Applies to:** Aurora PostgreSQL Provisioned clusters, RDS infrastructure, database networking, security configurations  
**Level:** Strategic/Tactical - critical for production database architecture  
**Audience:** Platform Engineers, Database Engineers, DevOps Engineers, Cloud Architects

## Core Principles

*Fundamental principles that guide all Aurora PostgreSQL design decisions, aligned with AWS Well-Architected Framework pillars.*

1. **Security First:** All database access must be encrypted, authenticated, and follow principle of least privilege with comprehensive audit logging
2. **High Availability by Design:** Multi-AZ deployment with automated failover, backup strategies, and disaster recovery capabilities built-in from day one
3. **Performance Optimization:** Right-sized instances with monitoring, parameter tuning, and connection pooling to ensure optimal database performance
4. **Cost Efficiency:** Resource optimization through appropriate instance sizing, storage optimization, and lifecycle management without compromising reliability
5. **Operational Excellence:** Infrastructure as Code with comprehensive monitoring, alerting, and automated operational procedures

## Rules

### Must Have (Critical)
*Non-negotiable rules that must always be followed. Violation of these rules should block deployment.*

- **RULE-001:** All Aurora clusters MUST be deployed across multiple Availability Zones (minimum 2 AZs) with automated failover enabled
- **RULE-002:** Encryption at rest MUST be enabled using AWS KMS with customer-managed keys for production environments
- **RULE-003:** Encryption in transit MUST be enforced with SSL/TLS connections required for all database connections
- **RULE-004:** Database clusters MUST be deployed in private subnets with no direct internet access
- **RULE-005:** All database credentials MUST be managed through AWS Secrets Manager with automatic rotation enabled
- **RULE-006:** Automated backups MUST be enabled with minimum 7-day retention period and point-in-time recovery capability
- **RULE-007:** Enhanced monitoring MUST be enabled with CloudWatch detailed monitoring and Performance Insights
- **RULE-008:** Database parameter groups MUST be custom (not default) with security and performance optimizations applied
- **RULE-009:** Deletion protection MUST be enabled for all production database clusters
- **RULE-010:** IAM database authentication MUST be enabled for administrative access where possible

### Should Have (Important)
*Strong recommendations that should be followed unless there's a compelling reason not to.*

- **RULE-101:** Implement database activity streams for audit logging and compliance requirements
- **RULE-102:** Use Aurora Serverless v2 scaling when workload patterns are variable or unpredictable
- **RULE-103:** Configure cross-region automated backups for disaster recovery in production environments
- **RULE-104:** Implement connection pooling (PgBouncer) for applications with high connection counts
- **RULE-105:** Use read replicas to distribute read workloads and improve performance
- **RULE-106:** Configure CloudWatch alarms for key metrics (CPU, connections, lag, storage)
- **RULE-107:** Implement database subnet groups spanning multiple AZs for network resilience
- **RULE-108:** Use AWS Config rules to continuously monitor Aurora configuration compliance
- **RULE-109:** Tag all Aurora resources comprehensively for cost tracking and resource management
- **RULE-110:** Implement automated testing of backup restoration procedures

### Could Have (Preferred)
*Best practices and preferences that improve quality but are not blocking.*

- **RULE-201:** Use Aurora Global Database for multi-region applications requiring low-latency global reads
- **RULE-202:** Implement custom CloudWatch dashboards for database performance monitoring
- **RULE-203:** Use AWS Systems Manager Parameter Store for non-sensitive database configuration
- **RULE-204:** Configure VPC Flow Logs for database subnet network monitoring
- **RULE-205:** Implement automated database maintenance window scheduling based on application usage patterns
- **RULE-206:** Use Aurora Machine Learning integration for advanced analytics workloads
- **RULE-207:** Configure database event subscriptions for proactive issue notification
- **RULE-208:** Implement database connection limit management based on instance class capabilities

## Patterns & Anti-Patterns

### ✅ Do This
*Concrete examples of proper Aurora PostgreSQL Terraform implementation*

```hcl
# Proper Aurora PostgreSQL cluster configuration
resource "aws_rds_cluster" "aurora_postgres" {
  cluster_identifier              = var.cluster_identifier
  engine                         = "aurora-postgresql"
  engine_version                 = "15.4"
  database_name                  = var.database_name
  master_username                = var.master_username
  manage_master_user_password    = true
  master_user_secret_kms_key_id  = aws_kms_key.aurora_key.arn
  
  # Security
  storage_encrypted               = true
  kms_key_id                     = aws_kms_key.aurora_key.arn
  
  # Networking
  db_subnet_group_name           = aws_db_subnet_group.aurora.name
  vpc_security_group_ids         = [aws_security_group.aurora.id]
  
  # Backup & Recovery
  backup_retention_period        = 30
  preferred_backup_window        = "03:00-04:00"
  preferred_maintenance_window   = "sun:04:00-sun:05:00"
  copy_tags_to_snapshot         = true
  deletion_protection           = true
  
  # Monitoring
  enabled_cloudwatch_logs_exports = ["postgresql"]
  
  # Parameter Group
  db_cluster_parameter_group_name = aws_rds_cluster_parameter_group.aurora_postgres.name
  
  tags = var.tags
}

resource "aws_rds_cluster_instance" "aurora_instances" {
  count                = var.instance_count
  identifier          = "${var.cluster_identifier}-${count.index + 1}"
  cluster_identifier  = aws_rds_cluster.aurora_postgres.id
  instance_class      = var.instance_class
  engine              = aws_rds_cluster.aurora_postgres.engine
  engine_version      = aws_rds_cluster.aurora_postgres.engine_version
  
  # Monitoring
  monitoring_interval = 60
  monitoring_role_arn = aws_iam_role.rds_enhanced_monitoring.arn
  performance_insights_enabled = true
  performance_insights_kms_key_id = aws_kms_key.aurora_key.arn
  
  tags = var.tags
}
```

### ❌ Don't Do This
*Concrete examples of anti-patterns to avoid*

```hcl
# Anti-pattern: Insecure Aurora configuration
resource "aws_rds_cluster" "bad_aurora" {
  cluster_identifier     = "my-db"
  engine                = "aurora-postgresql"
  master_username       = "postgres"
  master_password       = "password123"  # ❌ Hardcoded password
  
  storage_encrypted     = false          # ❌ No encryption
  backup_retention_period = 1            # ❌ Insufficient backup retention
  deletion_protection   = false          # ❌ No deletion protection
  skip_final_snapshot   = true           # ❌ No final snapshot
  
  # ❌ No security groups, monitoring, or parameter groups defined
}
```

## Decision Framework

*Guidance for making decisions when rules conflict or when faced with novel situations*

**When rules conflict:**
1. Security requirements always take precedence over cost optimization
2. High availability requirements supersede performance optimizations
3. Compliance and regulatory requirements override operational convenience

**When facing edge cases:**
- Consult AWS Well-Architected Framework documentation for Aurora PostgreSQL
- Engage with AWS Solutions Architects for complex architectural decisions
- Document all deviations from standard patterns with architectural decision records (ADRs)

## Exceptions & Waivers

*Define when and how these rules can be broken*

**Valid reasons for exceptions:**
- Development/testing environments (with security architect approval for relaxed security rules)
- Legacy system migrations with documented migration path to compliance
- Regulatory or compliance requirements that conflict with standard practices (with legal team approval)

**Process for exceptions:**
1. Document the exception, business justification, and risk assessment
2. Obtain approval from security architect and platform engineering lead
3. Implement compensating controls where possible
4. Schedule regular review (quarterly) with timeline for bringing into compliance

## Quality Gates

*Define how adherence to these rules should be verified*

- **Automated checks:** 
  - Terraform validation with custom policies using Sentinel or OPA
  - AWS Config rules for runtime compliance monitoring
  - Security scanning with tools like Checkov or Terrascan
  
- **Code review focus:** 
  - Security group configurations and network access patterns
  - Backup and disaster recovery strategy implementation
  - Monitoring and alerting configuration completeness
  
- **Testing requirements:** 
  - Automated backup restoration testing in non-production environments
  - Failover testing procedures documented and executed
  - Security testing including penetration testing for database access patterns

## Related Rules

*Reference other rules files that complement or interact with these rules*

- `rules/aws-networking-rules.md` - VPC and security group design patterns
- `rules/terraform-standards.md` - Infrastructure as Code best practices and standards
- `rules/monitoring-alerting-rules.md` - CloudWatch and observability requirements
- `rules/security-baseline-rules.md` - AWS security baseline and encryption standards

## References

*Links to external resources, standards, and documentation that inform these rules*

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) - Foundational design principles
- [Amazon Aurora PostgreSQL Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.BestPractices.html) - AWS official best practices
- [AWS RDS Security Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.Security.html) - Security configuration guidance
- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) - Resource configuration reference

---

## TL;DR

*Ultra-concise summary of the most critical rules and principles for Aurora PostgreSQL implementation.*

**Key Principles:**
- Security first with encryption everywhere and least privilege access
- High availability through multi-AZ deployment and comprehensive backup strategy
- Performance optimization with proper monitoring and resource sizing
- Cost efficiency without compromising reliability or security
- Operational excellence through Infrastructure as Code and automation

**Critical Rules:**
- Must deploy across multiple AZs with automated failover
- Must encrypt at rest and in transit with customer-managed KMS keys
- Must use private subnets with no direct internet access
- Must enable automated backups with minimum 7-day retention
- Must use AWS Secrets Manager for credential management

**Quick Decision Guide:**
When in doubt: Choose security and reliability over cost optimization, and always follow the principle of least privilege access.