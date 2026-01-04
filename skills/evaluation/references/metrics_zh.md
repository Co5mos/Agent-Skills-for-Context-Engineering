# 评估参考：指标与实现

本文档提供了评估指标和评估系统的实现细节。

## 核心指标定义

### 事实准确性 (Factual Accuracy)

衡量智能体输出中的声明是否与事实真相（Ground Truth）相符。

```
优秀 (1.0): 所有声明均经事实校验，无误。
良好 (0.8): 存在不影响主要结论的轻微错误。
及格 (0.6): 主要声明正确，但存在次要的不准确之处。
差 (0.3): 关键声明中存在显著的事实错误。
失败 (0.0): 存在导致输出失效的根本性事实错误。
```

计算方法：
- 从输出中提取声明。
- 对照事实真相验证每个声明。
- 根据重要性对声明进行加权（主要声明权重更高）。
- 计算声明准确性的加权平均值。

### 完整性 (Completeness)

衡量输出是否涵盖了所有请求的方面。

```
优秀 (1.0): 彻底涵盖了所有请求的方面。
良好 (0.8): 涵盖了绝大部分方面，存在细微遗漏。
及格 (0.6): 涵盖了关键方面，但存在一些缺漏。
差 (0.3): 输出中缺失了主要方面。
失败 (0.0): 未解决根本性的诉求。
```

### 引用准确性 (Citation Accuracy)

衡量引用的来源是否与声称的来源相符。

```
优秀 (1.0): 所有引用均准确且完整。
良好 (0.8): 存在细微的引用格式问题。
及格 (0.6): 主要引用准确。
差 (0.3): 存在显著的引用问题。
失败 (0.0): 引用缺失或完全错误。
```

### 来源质量 (Source Quality)

衡量是否使用了适当的一手来源。

```
优秀 (1.0): 使用权威的一手来源。
良好 (0.8): 大部分为一手来源，辅以少量二手来源。
及格 (0.6): 一手和二手来源混用。
差 (0.3): 大部分为二手或不可靠来源。
失败 (0.0): 未引用任何可信来源。
```

### 工具效率 (Tool Efficiency)

衡量智能体是否以合理的次数使用了适当的工具。

```
优秀 (1.0): 最优的工具选择和调用次数。
良好 (0.8): 工具选择良好，存在轻微的效率低下。
及格 (0.6): 使用了适当的工具，但存在一定的冗余。
差 (0.3): 使用了错误的工具或调用次数过多。
失败 (0.0): 严重的工具误用或极端冗余的调用。
```

## 评分表实现

```python
EVALUATION_DIMENSIONS = {
    "factual_accuracy": {
        "weight": 0.30,
        "description": "声明与事实真相相符",
        "levels": {
            "excellent": 1.0,
            "good": 0.8,
            "acceptable": 0.6,
            "poor": 0.3,
            "failed": 0.0
        }
    },
    "completeness": {
        "weight": 0.25,
        "description": "涵盖所有请求的方面",
        "levels": {
            "excellent": 1.0,
            "good": 0.8,
            "acceptable": 0.6,
            "poor": 0.3,
            "failed": 0.0
        }
    },
    "citation_accuracy": {
        "weight": 0.15,
        "description": "引用与来源相符",
        "levels": {
            "excellent": 1.0,
            "good": 0.8,
            "acceptable": 0.6,
            "poor": 0.3,
            "failed": 0.0
        }
    },
    "source_quality": {
        "weight": 0.10,
        "description": "使用了适当的一手来源",
        "levels": {
            "excellent": 1.0,
            "good": 0.8,
            "acceptable": 0.6,
            "poor": 0.3,
            "failed": 0.0
        }
    },
    "tool_efficiency": {
        "weight": 0.20,
        "description": "合理地使用了正确的工具",
        "levels": {
            "excellent": 1.0,
            "good": 0.8,
            "acceptable": 0.6,
            "poor": 0.3,
            "failed": 0.0
        }
    }
}

def calculate_overall_score(dimension_scores, rubric):
    """根据各维度分数计算加权总分。"""
    total_weight = 0
    weighted_sum = 0
    
    for dimension, score in dimension_scores.items():
        if dimension in rubric:
            weight = rubric[dimension]["weight"]
            weighted_sum += score * weight
            total_weight += weight
    
    return weighted_sum / total_weight if total_weight > 0 else 0
```

## 测试集管理

```python
class TestSet:
    def __init__(self, name):
        self.name = name
        self.tests = []
        self.tags = {}
    
    def add_test(self, test_case):
        """向测试集添加测试用例。"""
        self.tests.append(test_case)
        
        # 按标签索引
        for tag in test_case.get("tags", []):
            if tag not in self.tags:
                self.tags[tag] = []
            self.tags[tag].append(len(self.tests) - 1)
    
    def filter(self, **criteria):
        """按标准过滤测试。"""
        filtered = []
        for test in self.tests:
            match = True
            for key, value in criteria.items():
                if test.get(key) != value:
                    match = False
                    break
            if match:
                filtered.append(test)
        return filtered
    
    def get_complexity_distribution(self):
        """获取按复杂度划分的测试分布。"""
        distribution = {}
        for test in self.tests:
            complexity = test.get("complexity", "medium")
            distribution[complexity] = distribution.get(complexity, 0) + 1
        return distribution
```

## 评估运行器

```python
class EvaluationRunner:
    def __init__(self, test_set, rubric, agent):
        self.test_set = test_set
        self.rubric = rubric
        self.agent = agent
        self.results = []
    
    def run_all(self, verbose=False):
        """运行所有测试。"""
        self.results = []
        
        for i, test in enumerate(self.test_set.tests):
            if verbose:
                print(f"正在运行测试 {i+1}/{len(self.test_set.tests)}")
            
            result = self.run_test(test)
            self.results.append(result)
        
        return self.summarize()
    
    def run_test(self, test):
        """运行单个评估测试。"""
        # 获取智能体输出
        output = self.agent.run(test["input"])
        
        # 评估
        evaluation = self.evaluate_output(output, test)
        
        return {
            "test": test,
            "output": output,
            "evaluation": evaluation
        }
    
    def evaluate_output(self, output, test):
        """根据测试评估智能体输出。"""
        ground_truth = test.get("expected", {})
        
        dimension_scores = {}
        for dimension, config in self.rubric.items():
            score = self.evaluate_dimension(
                output, ground_truth, dimension, config
            )
            dimension_scores[dimension] = score
        
        overall = calculate_overall_score(dimension_scores, self.rubric)
        
        return {
            "overall_score": overall,
            "dimension_scores": dimension_scores,
            "passed": overall >= 0.7
        }
    
    def summarize(self):
        """总结评估结果。"""
        if not self.results:
            return {"error": "无结果"}
        
        passed = sum(1 for r in self.results if r["evaluation"]["passed"])
        
        dimension_totals = {}
        for dimension in self.rubric.keys():
            dimension_totals[dimension] = {
                "total": 0,
                "count": 0
            }
        
        for result in self.results:
            for dimension, score in result["evaluation"]["dimension_scores"].items():
                if dimension in dimension_totals:
                    dimension_totals[dimension]["total"] += score
                    dimension_totals[dimension]["count"] += 1
        
        dimension_averages = {}
        for dimension, data in dimension_totals.items():
            if data["count"] > 0:
                dimension_averages[dimension] = data["total"] / data["count"]
        
        return {
            "total_tests": len(self.results),
            "passed": passed,
            "failed": len(self.results) - passed,
            "pass_rate": passed / len(self.results) if self.results else 0,
            "dimension_averages": dimension_averages,
            "failures": [
                r for r in self.results 
                if not r["evaluation"]["passed"]
            ]
        }
```

## 生产环境监控

```python
class ProductionMonitor:
    def __init__(self, sample_rate=0.01):
        self.sample_rate = sample_rate
        self.samples = []
        self.alert_thresholds = {
            "pass_rate_warning": 0.85,
            "pass_rate_critical": 0.70
        }
    
    def sample_and_evaluate(self, query, output):
        """采样生产环境交互进行评估。"""
        if random.random() > self.sample_rate:
            return None
        
        evaluation = evaluate_output(output, {}, EVALUATION_RUBRIC)
        
        sample = {
            "query": query[:200],
            "output_preview": output[:200],
            "score": evaluation["overall_score"],
            "passed": evaluation["passed"],
            "timestamp": current_timestamp()
        }
        
        self.samples.append(sample)
        return sample
    
    def get_metrics(self):
        """从采样中计算当前指标。"""
        if not self.samples:
            return {"status": "数据不足"}
        
        passed = sum(1 for s in self.samples if s["passed"])
        pass_rate = passed / len(self.samples)
        
        avg_score = sum(s["score"] for s in self.samples) / len(self.samples)
        
        return {
            "sample_count": len(self.samples),
            "pass_rate": pass_rate,
            "average_score": avg_score,
            "status": self._get_status(pass_rate)
        }
    
    def _get_status(self, pass_rate):
        """根据通过率获取状态。"""
        if pass_rate < self.alert_thresholds["pass_rate_critical"]:
            return "critical (紧急)"
        elif pass_rate < self.alert_thresholds["pass_rate_warning"]:
            return "warning (警告)"
        else:
            return "healthy (健康)"
```
