# 接口自动化测试框架

基于 Python + requests + pytest 的接口自动化测试框架

## 框架特性

- ✅ 支持运营端登录获取token
- ✅ 统一的API客户端封装
- ✅ 配置文件管理（YAML格式）
- ✅ 测试数据管理和持久化
- ✅ 完善的断言和验证机制
- ✅ 支持多种测试报告（HTML、Allure）
- ✅ 清晰的测试用例结构

## 项目结构

```
api-test-framework/
├── config/                    # 配置文件目录
│   ├── base_config.yaml      # 基础配置（环境、账号等）
│   └── test_data.yaml        # 测试数据（运行时生成）
├── common/                    # 公共模块
│   ├── __init__.py
│   ├── api_client.py         # API客户端封装
│   ├── auth.py               # 认证管理
│   ├── data_loader.py        # 数据加载器
│   └── utils.py              # 工具函数
├── tests/                     # 测试用例目录
│   ├── __init__.py
│   ├── test_flow_000.py      # FLOW-000测试用例
│   ├── test_flow_001.py      # FLOW-001测试用例
│   ├── test_flow_002.py      # FLOW-002测试用例
│   ├── test_flow_003.py      # FLOW-003测试用例
│   └── test_flow_004.py      # FLOW-004测试用例
├── reports/                   # 测试报告目录（自动生成）
│   ├── html/                 # HTML报告
│   └── allure/               # Allure报告
├── conftest.py               # pytest配置和fixtures
├── pytest.ini                # pytest配置文件
├── requirements.txt          # 依赖包
└── README.md                 # 本文件
```

## 快速开始

### 1. 安装依赖

```bash
pip install -r requirements.txt
```

### 2. 配置参数

编辑 `config/base_config.yaml` 文件，填写以下配置：

```yaml
# 运营端登录账号
ops_account:
  username:# TODO: 填写实际的用户名
  password:# TODO: 填写实际的密码

# 测试数据配置（可选）
test_data:
  test_company_id: null  # 如果有已知的公司ID，可以填写
  media_platform: facebook  # 媒体平台
```

### 3. 运行测试

#### 运行单个测试用例

```bash
# 运行FLOW-000测试用例
pytest tests/test_flow_000.py -v -s

# 运行特定步骤
pytest tests/test_flow_000.py::TestFlow002UserManagement::test_step1_get_company_list -v -s
```

#### 运行所有测试

```bash
# 运行所有测试
pytest -v

# 运行P0级别的测试
pytest -m p0 -v

# 运行流程测试
pytest -m flow -v
```

#### 生成报告

```bash
# 生成HTML报告（自动生成）
pytest --html=reports/html/report.html --self-contained-html

# 生成Allure报告
pytest --alluredir=reports/allure
allure serve reports/allure  # 查看Allure报告
```

## 配置说明

### base_config.yaml

主要配置项：

- `environments.ops_login_url`: 运营端登录地址
- `environments.api_base_url`: API基础地址
- `environments.timeout`: 请求超时时间
- `ops_account.username`: 登录用户名
- `ops_account.password`: 登录密码
- `test_data.media_platform`: 测试使用的媒体平台

### test_data.yaml

运行时自动生成，用于存储：
- 当前登录token
- 测试过程中产生的临时数据（如用户ID、账户ID等）
- 需要清理的数据标识

## 测试用例编写示例

```python
import pytest
from common.api_client import APIClient

@pytest.mark.p0
class TestExample:
    def test_example(self, api_client: APIClient):
        """示例测试用例"""
        # 发送请求
        result = api_client.request_with_assert(
            method='POST',
            endpoint='/api/example',
            json={"key": "value"},
            expected_code=200,
            expected_biz_code=0
        )
        
        # 验证响应
        assert result['code'] == 0
        assert 'data' in result
```

## 常用功能

### API客户端使用

```python
# 使用fixture自动注入
def test_example(api_client: APIClient):
    # GET请求
    result = api_client.get('/api/endpoint', params={'key': 'value'})
    
    # POST请求
    result = api_client.post('/api/endpoint', json={'key': 'value'})
    
    # 带断言的请求
    result = api_client.request_with_assert(
        method='POST',
        endpoint='/api/endpoint',
        json={'key': 'value'},
        expected_code=200,
        expected_biz_code=0
    )
```

### 数据管理

```python
from common.data_loader import data_loader

# 获取配置
base_url = data_loader.get_config('environments.api_base_url')

# 获取/设置测试数据
user_id = data_loader.get_test_data('test_context.user_id')
data_loader.set_test_data('test_context.user_id', 123)
```

### 工具函数

```python
from common.utils import (
    generate_test_email,      # 生成测试邮箱
    assert_response_code,     # 断言响应状态码
    validate_response_structure  # 验证响应结构
)
```

## 注意事项

1. **登录配置**: 首次使用前需要在 `base_config.yaml` 中配置正确的登录账号
2. **数据清理**: 测试完成后会自动清理测试数据（如果实现了清理接口）
3. **响应结构**: 根据实际接口响应结构调整验证逻辑
4. **Token管理**: Token会自动获取和缓存，无需手动管理
5. **测试隔离**: 每个测试用例尽量独立，避免相互依赖

## 扩展说明

### 添加新的测试用例

1. 在 `tests/` 目录下创建新的测试文件
2. 参考 `test_flow_000.py` 或其他测试用例的写法
3. 使用 `api_client` fixture 发送请求
4. 添加适当的断言和验证

### 添加新的工具函数

在 `common/utils.py` 中添加新的工具函数，供所有测试用例使用。

### 自定义报告

可以通过修改 `pytest.ini` 配置自定义报告格式和内容。

## 问题排查

### 登录失败

1. 检查 `base_config.yaml` 中的账号密码是否正确
2. 检查登录接口URL是否正确
3. 检查网络连接是否正常
4. 查看控制台输出的错误信息

### 接口调用失败

1. 检查API地址配置是否正确
2. 检查token是否有效
3. 查看响应内容确认错误原因
4. 检查请求参数格式是否正确

### 测试数据问题

1. 检查前置步骤是否执行成功
2. 查看 `config/test_data.yaml` 确认数据是否正确保存
3. 确认测试数据是否满足接口要求

## 联系方式

如有问题，请联系测试团队。

---

**版本**: v1.0  
**最后更新**: 2025-12-27

