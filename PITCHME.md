# Unit tests and integration tests in python
## Introduction
This presentation consists of the following parts
* General concepts - A "no code allowed" introduction to define some commonly used terms.
* Unit testing in python
* Integration testing in python

---

## General concepts
### what is a unit test?
* A unit test is a way of testing a unit, the smallest piece of code that can be logically isolated in a system. In most programming languages, that is a function or a method.
* The above definition is the most common definition but not the original one. Originally in SUnit (Smalltalk unit) unit revered to the independence of the test. i.e. test-1 did not influence test-2.

---

## General concepts
### Why?
* A common misconception is that unit tests are for finding bugs. 
* A large coverage does not say much about code quality either.
* Unit tests is a design tool that helps developer to write code in small, single usage methods.
* This is why tests should be written before the code. (TDD)

---

## General concepts
### What is an integration test?
There are at least two different notions of what integration tests are.
* Narrow integration tests definition
* * Tests only that portion of the code in my service that talks to a third party service
* * Uses mocks of those third party services, either in process or remote
thus consist of many narrowly scoped tests, often no larger in scope than a unit test (and usually run with the same test framework that's used for unit tests)

---

## General concepts
### What is a integration test?
* Broad integration tests definition
* * Require live versions of all services therefore requiring substantial test infrastructure to mimic the production environment.
* * Exercise code paths through all services, not just "our code" responsible for interactions with third party API's

---

## General concepts
### What is a integration test?
A large part of the software developer community only know about the broad kind of integration test which can lead to lots of confusion when other developers use the narrow definition. Therefore the broad definitionn of integration tests are often called end to end tests. 

At this moment we (Inventory manager) only use the narrow integration tests. It is definitely interesting to use end-to-end testing in the future but we are not there yet.

---

## General concepts
### What is a mock?
* Mocks are used in the narrow definition of integration tests.
* In object-oriented programming, mock objects are simulated objects that mimic the behavior of real objects in controlled ways. (Wikipedia)
* Mock objects have the same interface as the real objects they mimic, allowing a client object (the object we are trying to test) to remain unaware of whether it is using a real object or a mock object.

---

## General concepts
### writing testable code
* methods should be short <-- also makes them easier to read
* methods should do only one thing
* methods should have no side effects if possible

---

## General concepts
### What to test?
* TDD recommends if you write new code or refactor existing code you write a unit tests as it helps you write better more maintanable code. Don't unit test legacy code.
* Best thing to do is follow true TDD. Don't test for testing's sake. Test as design technique.
* Test because it enforces loosely coupled code. 
* Test because having code executed in two contexts (test and application) makes better code. 
* Once you go down the path of TDD you don't ask what you should and shouldn't test. Testing just becomes part of writing code and not an independent activity.

---

## General concepts
### What to test?
* what to test with integration tests (answer everything)
* * Everything means from a functional perspective. Every business use needs one integration test.
* Code not covered by integration test can mean two things
* * You need more tests
* * You need to remove code

---

## Unit testing in python
### Installation
```
conda install -c anaconda pytest
conda install -c anaconda pytest-mock
conda install -c anaconda pytest-runner
conda install -c anaconda pytest-cov
```

---

## Unit testing in python
### Running tests manually
```bash
cd test
# To run all tests
pytest -s -x

# To run the tests of one specific file
pytest -s -x test_price_integration.py

# -s shows more on screen (optional)
# -x stop on first error (optional)
```

---

## Unit testing in python
### Creating unit tests
* Name of test file should start with 'test_' i.e. test_price.py
* The name of each test method should start with test_ 
* make sure the test script can find the code.
```
import sys
sys.path.append('../code/sf_pricing')
```

---

## Unit testing in python
### Creating unit tests
* Objects that need to be there for every test use a fixture annotation
```
@pytest.fixture
def init_pricer():
    """initialize a pricer"""
    options = CommandLineOptions('GB00B03MLX29', 'XAMS', 'UAT', None, None, None, '2017-08-09')
    pricer = Pricer(options.env, options.mrkt, options.isin, options.test, options.day)
    return pricer


def test_set_day(init_pricer):
    assert init_pricer._set_day() == date.today()    # Test no input, should put it at current day
    assert init_pricer._set_day('2016-09-12') == date(2016, 9, 12)  # Test Arbitrary past date
```
* test more than only happy flow

---

## Integration testing in python
### How to write a integration test
* Run integration test with fixed input
```
class CommandLineOptions:
    def __init__(self, isin, mrkt, env, test, value, day, curdate, mode):
        self.isin = isin
 		... # Other options omited for brevity


def run_test():
    options = CommandLineOptions('GB00B03MLX29', 'XAMS', 'UAT', None, None, None, '2017-08-09', 'cur')
    pricer = Pricer(options.env, options.mrkt, options.isin, options.test, options.day, options.mode)
    pricer.price()

@mock.patch('localdatahub.InventoryService', testdatahub.MockInventoryService11)
@mock.patch('pythonODBCImpala.DatahubConnectionService', test_ODBC.MockDatahubConnectionService11)
def test_price_integration_1_1():
    run_test()
    datestring = datetime.today().strftime('%Y%m%d')
    line1 = f"GB00B03MLX29|XAMS|{datestring}|SETTLED|"
    line2 = f"FR0000031122|XAMS|{datestring}|SETTLED|"
    check_datafile(line1, line2)
```

---

## Integration testing in python
### How to write a mock
* mocks are dead simple
localdatahub.py
```
class InventoryService:
	def request(self, isin, mnemonic, days, style, trans=False):
		# bunch of code that among other things uses external dependency
	# bunch of other methods
```
testdatahub.py
```
def generic_request(step, style):
    fn = os.path.join(os.path.dirname(__file__), f"testdata/{step}/{style}Responses.json")
    f = open(fn, 'r')
    resp = f.read()
    f.close()
    resp_data = json.loads(resp)
    return resp_data


class MockInventoryService11(InventoryService):
    def request(self, isin, mnemonic, days, style, trans=False):
        return generic_request(1.1, style)
```

---

The future
continuous integration
BDD

## References
* ![Martin Fowler on integration tests](https://martinfowler.com/bliki/IntegrationTest.html)
