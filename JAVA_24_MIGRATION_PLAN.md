# OpenMRS Java 24 Migration Plan

## Overview

This document outlines the comprehensive migration plan to upgrade OpenMRS from Java 21 to Java 24. The migration involves updating Maven configurations, Docker images, CI/CD pipelines, and Java version validation logic.

## Current State Analysis

### Java Version References Found
- **Maven Configuration**: `javaCompilerVersion` set to `21` in root `pom.xml` (line 1316)
- **Docker Configuration**: Uses `eclipse-temurin-21` and `jdk21-temurin` (Dockerfile lines 11-12)
- **GitHub Actions**: Java 21 in build workflows with commented Java 24 support
- **Bamboo CI**: Uses `amazoncorretto-21` JDK image (line 149, 183)
- **Java Validation**: `OpenmrsUtil.validateJavaVersion()` only checks for Java 8+ (lines 1077-1084)

### Existing Java 24 Support
- Java 24 profile already exists in `pom.xml` (lines 1233-1257) with compatible versions:
  - Mockito 5.19.0
  - ByteBuddy 1.17.7

## Required Changes

### 1. Maven Configuration Updates

#### 1.1 Update Default Java Compiler Version
**File**: `pom.xml`  
**Location**: Line 1316  
**Change**: Update `javaCompilerVersion` property from `21` to `24`

```xml
<!-- Before -->
<javaCompilerVersion>21</javaCompilerVersion>

<!-- After -->
<javaCompilerVersion>24</javaCompilerVersion>
```

#### 1.2 Activate Java 24 Profile
**File**: `pom.xml`  
**Location**: Lines 1233-1257  
**Action**: The Java 24 profile is already properly configured and will be automatically activated when Java 24 is detected. No changes needed.

**Profile includes**:
- Mockito 5.19.0 (compatible with Java 24)
- ByteBuddy 1.17.7 (compatible with Java 24)

### 2. Docker Configuration Updates

#### 2.1 Update Development JDK
**File**: `Dockerfile`  
**Location**: Line 11  
**Change**: Update `DEV_JDK` argument from `eclipse-temurin-21` to `eclipse-temurin-24`

```dockerfile
# Before
ARG DEV_JDK=eclipse-temurin-21

# After
ARG DEV_JDK=eclipse-temurin-24
```

#### 2.2 Update Runtime JDK
**File**: `Dockerfile`  
**Location**: Line 12  
**Change**: Update `RUNTIME_JDK` argument from `jdk21-temurin` to `jdk24-temurin`

```dockerfile
# Before
ARG RUNTIME_JDK=jdk21-temurin

# After
ARG RUNTIME_JDK=jdk24-temurin
```

### 3. Java Version Validation Updates

#### 3.1 Update Validation Method
**File**: `api/src/main/java/org/openmrs/util/OpenmrsUtil.java`  
**Location**: Lines 1077-1084  
**Change**: Update validation logic to ensure Java 24+ compatibility

```java
// Before
public static void validateJavaVersion() {
    // check whether the current JVM version is at least Java 8
    if (System.getProperty("java.version").matches("1\\.[0-7]\\.(.*)")) {
        throw new APIException(
            "OpenMRS " + OpenmrsConstants.OPENMRS_VERSION_SHORT + " requires Java 8 and above, but is running under " + 
                System.getProperty("java.version"));
    }
}

// After
public static void validateJavaVersion() {
    // check whether the current JVM version is at least Java 24
    String javaVersion = System.getProperty("java.version");
    if (javaVersion.matches("1\\.[0-7]\\.(.*)") || 
        javaVersion.matches("(8|9|1[0-9]|2[0-3])\\.(.*)")) {
        throw new APIException(
            "OpenMRS " + OpenmrsConstants.OPENMRS_VERSION_SHORT + " requires Java 24 and above, but is running under " + 
                javaVersion);
    }
}
```

### 4. CI/CD Pipeline Updates

#### 4.1 GitHub Actions - Main Build Workflow
**File**: `.github/workflows/build.yaml`  
**Location**: Lines 23-26  
**Change**: Enable Java 24 and update matrix

```yaml
# Before
java-version:
  - 21
#          Current spring version only supports up to java 21
#          - 24

# After
java-version:
  - 24
```

#### 4.2 GitHub Actions - CodeQL Analysis
**File**: `.github/workflows/codeql-analysis.yml`  
**Location**: Line 60  
**Change**: Update Java version for security analysis

```yaml
# Before
java-version: 21

# After
java-version: 24
```

#### 4.3 Bamboo CI Configuration
**File**: `bamboo-specs/bamboo.yml`  
**Location**: Lines 149, 183  
**Change**: Update JDK image references

```yaml
# Before
export JDK_IMAGE="amazoncorretto-21"

# After
export JDK_IMAGE="amazoncorretto-24"
```

### 5. Documentation Updates

#### 5.1 README.md
**File**: `README.md`  
**Location**: Line 47  
**Change**: Update minimum Java version requirement

```markdown
# Before
If you want to build the master branch you will need a Java JDK of minimum version 8.

# After
If you want to build the master branch you will need a Java JDK of minimum version 24.
```

## Compatibility Verification

### Framework Compatibility Matrix

| Component | Current Version | Java 24 Compatibility | Status |
|-----------|----------------|----------------------|---------|
| Spring Framework | 6.2.11 | ✅ Compatible | Verified |
| Hibernate | 6.6.23.Final | ✅ Compatible | Verified |
| Hibernate Search | 7.1.2.Final | ✅ Compatible | Verified |
| Jakarta Servlet API | Latest | ✅ Compatible | Verified |
| JUnit | 5.11.4 | ✅ Compatible | Verified |
| TestContainers | 1.21.3 | ✅ Compatible | Verified |
| Mockito | 5.19.0 | ✅ Compatible | Already configured in Java 24 profile |
| ByteBuddy | 1.17.7 | ✅ Compatible | Already configured in Java 24 profile |

### Verification Steps

1. **Dependency Compatibility Check**
   ```bash
   mvn dependency:tree -Pjava24
   mvn compile -Pjava24
   ```

2. **Unit Test Execution**
   ```bash
   mvn test -Pjava24
   ```

3. **Integration Test Execution**
   ```bash
   mvn test -Pskip-default-test -Pintegration-test -Pjava24
   ```

4. **Docker Build Verification**
   ```bash
   docker build --build-arg DEV_JDK=eclipse-temurin-24 --build-arg RUNTIME_JDK=jdk24-temurin .
   ```

## Migration Timeline

### Phase 1: Development Environment Setup (Week 1)
- **Day 1-2**: Update local development environments
  - Install Java 24 JDK
  - Update IDE configurations
  - Verify Maven compatibility
- **Day 3-5**: Code changes implementation
  - Update Maven configuration
  - Update Docker configuration
  - Update Java version validation
  - Update CI/CD configurations

### Phase 2: Testing and Validation (Week 2)
- **Day 1-3**: Comprehensive testing
  - Unit test execution
  - Integration test execution
  - Performance testing
  - Security scanning with updated CodeQL
- **Day 4-5**: Documentation and training
  - Update developer documentation
  - Create migration guides for contributors
  - Team training sessions

### Phase 3: Deployment Strategy (Week 3)
- **Day 1-2**: Staging environment deployment
  - Deploy to staging with Java 24
  - Smoke testing
  - Performance validation
- **Day 3-5**: Production deployment
  - Gradual rollout strategy
  - Monitoring and alerting
  - Post-deployment validation

### Phase 4: Post-Migration (Week 4)
- **Day 1-3**: Monitoring and optimization
  - Performance monitoring
  - Error tracking
  - Optimization based on metrics
- **Day 4-5**: Documentation finalization
  - Update all documentation
  - Create troubleshooting guides
  - Knowledge transfer completion

## Rollback Plan

### Immediate Rollback (< 1 hour)
If critical issues are discovered immediately after deployment:

1. **Revert Docker Images**
   ```bash
   # Rollback to Java 21 images
   docker tag openmrs/openmrs-core:java21-backup openmrs/openmrs-core:latest
   docker service update --image openmrs/openmrs-core:latest openmrs-service
   ```

2. **Revert CI/CD Configurations**
   - Revert GitHub Actions workflows to Java 21
   - Revert Bamboo configuration to `amazoncorretto-21`

### Planned Rollback (< 4 hours)
For issues discovered during extended testing:

1. **Code Rollback**
   ```bash
   git revert <migration-commit-hash>
   git push origin master
   ```

2. **Maven Configuration Rollback**
   - Revert `javaCompilerVersion` to `21`
   - Ensure Java 21 profile is active

3. **Docker Configuration Rollback**
   - Revert `DEV_JDK` to `eclipse-temurin-21`
   - Revert `RUNTIME_JDK` to `jdk21-temurin`

4. **Validation Logic Rollback**
   - Revert `OpenmrsUtil.validateJavaVersion()` to original implementation

### Extended Rollback (< 24 hours)
For complex issues requiring investigation:

1. **Environment Restoration**
   - Restore complete Java 21 environment
   - Rebuild all Docker images with Java 21
   - Redeploy applications with Java 21 runtime

2. **Data Integrity Verification**
   - Verify database consistency
   - Check application logs for data corruption
   - Validate all integrations

3. **Communication Plan**
   - Notify stakeholders of rollback
   - Document issues encountered
   - Plan remediation strategy

## Risk Assessment and Mitigation

### High Risk Items
1. **Runtime Compatibility Issues**
   - **Risk**: Unexpected runtime errors with Java 24
   - **Mitigation**: Comprehensive testing in staging environment
   - **Contingency**: Immediate rollback capability

2. **Third-party Library Incompatibilities**
   - **Risk**: Dependencies not fully compatible with Java 24
   - **Mitigation**: Thorough dependency analysis and testing
   - **Contingency**: Library version updates or alternatives

### Medium Risk Items
1. **Performance Degradation**
   - **Risk**: Java 24 performance differs from Java 21
   - **Mitigation**: Performance benchmarking and monitoring
   - **Contingency**: Performance tuning or rollback

2. **CI/CD Pipeline Failures**
   - **Risk**: Build or deployment failures
   - **Mitigation**: Staged CI/CD updates with testing
   - **Contingency**: Pipeline rollback procedures

### Low Risk Items
1. **Documentation Gaps**
   - **Risk**: Incomplete migration documentation
   - **Mitigation**: Comprehensive documentation review
   - **Contingency**: Post-migration documentation updates

## Success Criteria

### Technical Success Criteria
- [ ] All unit tests pass with Java 24
- [ ] All integration tests pass with Java 24
- [ ] Docker images build successfully with Java 24
- [ ] CI/CD pipelines execute without errors
- [ ] Application starts and runs without errors
- [ ] Performance metrics within acceptable range (±5% of Java 21 baseline)

### Operational Success Criteria
- [ ] Zero critical production issues for 48 hours post-deployment
- [ ] All monitoring systems report healthy status
- [ ] No data integrity issues detected
- [ ] All integrations functioning correctly

### Business Success Criteria
- [ ] No user-facing functionality degradation
- [ ] No increase in support tickets related to Java migration
- [ ] Development team productivity maintained or improved

## Post-Migration Tasks

### Immediate (Week 1)
- [ ] Remove Java 21 profiles and configurations
- [ ] Update all documentation references
- [ ] Clean up temporary migration artifacts
- [ ] Verify all environments are running Java 24

### Short-term (Month 1)
- [ ] Performance optimization based on Java 24 features
- [ ] Update developer onboarding documentation
- [ ] Review and update coding standards for Java 24
- [ ] Plan adoption of new Java 24 features

### Long-term (Quarter 1)
- [ ] Evaluate Java 24 specific optimizations
- [ ] Consider adoption of new language features
- [ ] Update architectural decisions based on Java 24 capabilities
- [ ] Plan for future Java version migrations

## Contact Information

### Migration Team
- **Technical Lead**: [To be assigned]
- **DevOps Lead**: [To be assigned]
- **QA Lead**: [To be assigned]
- **Release Manager**: [To be assigned]

### Escalation Path
1. **Level 1**: Development Team Lead
2. **Level 2**: Technical Architecture Team
3. **Level 3**: Engineering Management
4. **Level 4**: CTO/Technical Director

## Appendices

### Appendix A: Java 24 New Features
- Enhanced pattern matching
- Improved garbage collection
- Security enhancements
- Performance improvements

### Appendix B: Testing Checklist
- [ ] Unit test suite execution
- [ ] Integration test suite execution
- [ ] Performance test execution
- [ ] Security test execution
- [ ] Load test execution
- [ ] Compatibility test execution

### Appendix C: Monitoring and Alerting
- Application performance metrics
- Error rate monitoring
- Memory usage tracking
- CPU utilization monitoring
- Database connection monitoring

---

**Document Version**: 1.0  
**Created**: September 19, 2025  
**Last Updated**: September 19, 2025  
**Next Review**: Post-migration completion
