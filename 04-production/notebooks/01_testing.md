# Production 01: Testing with Pytest

Testing is non-negotiable for production code. Pytest is the de-facto standard for Python testing.

## 1. Why Pytest?
- Simple syntax (`assert` statements).
- Powerful fixtures for setup/teardown.
- Rich plugin ecosystem.

## 2. Basic Test Structure

```python
# test_calculator.py
def test_add():
    assert 1 + 1 == 2

def test_subtract():
    assert 5 - 3 == 2
```

Run with: `pytest test_calculator.py`

## 3. Fixtures
Reusable setup code.

```python
import pytest

@pytest.fixture
def sample_user():
    return {"name": "Alice", "age": 30}

def test_user_name(sample_user):
    assert sample_user["name"] == "Alice"
```

## 4. Parametrize (Multiple Test Cases)

```python
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (5, 5, 10),
    (-1, 1, 0),
])
def test_add(a, b, expected):
    assert a + b == expected
```

## 5. Mocking
Replace external dependencies with fakes.

```python
from unittest.mock import Mock

def test_api_call(mocker):
    mock_response = Mock()
    mock_response.status_code = 200
    mocker.patch('requests.get', return_value=mock_response)
    # Now requests.get() returns our mock
```
