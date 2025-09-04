# Commit Convention Guide

## Quy tắc Commit Message

Sử dụng format: `<type>(<scope>): <description>`

### Types

- **feat**: Thêm tính năng mới
- **fix**: Sửa lỗi
- **docs**: Thay đổi tài liệu
- **style**: Thay đổi format code (không ảnh hưởng logic)
- **refactor**: Refactor code (không thêm tính năng hoặc sửa lỗi)
- **test**: Thêm hoặc sửa test
- **chore**: Thay đổi build process, dependencies, tools
- **perf**: Cải thiện performance
- **ci**: Thay đổi CI/CD configuration
- **build**: Thay đổi build system hoặc dependencies
- **revert**: Revert commit trước đó

### Scope (Optional)

Tên module/service được thay đổi:
- `user-service`
- `talent-pool-service`
- `business-service`
- `contract-service`
- `odc-team-service`
- `project-service`
- `notification-service`
- `analytics-service`
- `skill-matching-service`
- `marketplace-service`
- `learning-service`
- `api-gateway`
- `config-server`
- `eureka-server`
- `shared-lib`
- `database`
- `deployment`

### Description

- Sử dụng tiếng Anh
- Bắt đầu bằng động từ ở dạng nguyên thể
- Không viết hoa chữ cái đầu
- Không kết thúc bằng dấu chấm
- Tối đa 50 ký tự

## Ví dụ Commit Messages

### Feature Development
```
feat(user-service): add user registration endpoint
feat(talent-pool-service): implement skill assessment feature
feat(contract-service): add automated contract generation
feat(api-gateway): implement rate limiting middleware
feat(shared-lib): add common exception handling utilities
```

### Bug Fixes
```
fix(user-service): resolve JWT token validation issue
fix(business-service): fix null pointer exception in profile update
fix(database): correct foreign key constraint in contracts table
fix(deployment): resolve Docker container startup issue
```

### Documentation
```
docs: update API documentation for user endpoints
docs(readme): add setup instructions for development environment
docs(architecture): update microservices communication diagram
```

### Refactoring
```
refactor(user-service): extract validation logic to separate class
refactor(shared-lib): simplify response wrapper implementation
refactor(contract-service): optimize database query performance
```

### Testing
```
test(user-service): add unit tests for authentication controller
test(talent-pool-service): implement integration tests for skill matching
test(e2e): add end-to-end tests for contract workflow
```

### Chores
```
chore: update Spring Boot version to 3.2.0
chore(deps): bump jackson-databind from 2.15.0 to 2.15.2
chore(config): update application properties for production
chore(docker): optimize Dockerfile for faster builds
```

### Performance
```
perf(analytics-service): optimize data aggregation queries
perf(api-gateway): implement response caching
perf(database): add indexes for frequently queried columns
```

### CI/CD
```
ci: add automated testing pipeline
ci(jenkins): update deployment script for staging environment
ci(docker): add multi-stage build for production images
```

### Build
```
build: update Maven dependencies
build(gradle): configure multi-module project structure
build(docker-compose): add Redis and Kafka services
```

### Style
```
style(user-service): format code according to Google Java Style
style: fix indentation and remove trailing whitespaces
```

### Revert
```
revert: "feat(user-service): add experimental caching feature"
```

## Breaking Changes

Nếu commit có breaking changes, thêm `BREAKING CHANGE:` vào footer:

```
feat(api-gateway): change authentication flow

BREAKING CHANGE: API endpoints now require Bearer token instead of Basic auth
```

## Multi-line Commits

Đối với commits phức tạp, sử dụng body để giải thích:

```
feat(contract-service): implement digital signature workflow

- Add DocuSign integration for contract signing
- Implement signature validation and verification
- Add audit trail for signature events
- Update contract status based on signature completion

Closes #123
```

## Quy tắc Bổ sung

1. **Atomic commits**: Mỗi commit chỉ thực hiện một thay đổi logic
2. **Frequent commits**: Commit thường xuyên, tránh commits quá lớn
3. **Meaningful messages**: Message phải mô tả rõ ràng thay đổi
4. **Reference issues**: Sử dụng `Closes #issue-number` khi fix issue
5. **Review before commit**: Kiểm tra code trước khi commit

## Tools Hỗ trợ

### Commitizen
```bash
npm install -g commitizen
npm install -g cz-conventional-changelog
echo '{"path": "cz-conventional-changelog"}' > ~/.czrc
```

Sử dụng: `git cz` thay vì `git commit`

### Conventional Commits VSCode Extension
Cài đặt extension "Conventional Commits" trong VSCode để hỗ trợ tạo commit message.

### Git Hooks
Thêm pre-commit hook để validate commit message format:

```bash
#!/bin/sh
# .git/hooks/commit-msg
commit_regex='^(feat|fix|docs|style|refactor|test|chore|perf|ci|build|revert)(\(.+\))?: .{1,50}'

if ! grep -qE "$commit_regex" "$1"; then
    echo "Invalid commit message format!"
    echo "Use: <type>(<scope>): <description>"
    exit 1
fi
```