代码已完成，请为/xx/xxx/xxx 接口编写测试用例，要求如下：

## 文件结构

| 类型 | 路径模板 | 示例 |
|------|----------|------|
| 测试脚本 | `{project_name}/tests/test_{interface_name}.py` | `tests/test_deep_optimization_converse.py` |
| 请求体 | `{project_name}/tests/test_data/{interface_name}/scenario_{num}_{scenario}.json` | `tests/test_data/deep_optimization_converse/scenario_1_first_call.json` |
| 输出结果 | `{project_name}/tests/output/{interface_name}/scenario_{num}_{scenario}_output.txt` | `tests/output/deep_optimization_converse/scenario_1_first_call_output.txt` |

## 测试脚本要求

脚本顶部定义两个配置变量：
API_URL = "..."      # 固定接口地址
INPUT_FILE = "..."   # 输入JSON文件路径，用于切换不同测试场景## 测试数据要求

- 提供 **3-5 个** 覆盖不同场景的请求体JSON文件
- LLM返回结果持久化到对应的txt文件，便于后续审查校对