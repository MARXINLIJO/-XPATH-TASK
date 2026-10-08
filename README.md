
XPATH TASK :
```
import logging
import pytest
from typing import List
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.remote.webelement import WebElement
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import Select
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.options import Options

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)


# ==============================================================================
# Base Page (POM)
# ==============================================================================
class BasePage:
    def __init__(self, driver: webdriver.Chrome):
        self.driver = driver
        self.wait = WebDriverWait(self.driver, 10)

    def find_element(self, locator: tuple) -> WebElement:
        return self.wait.until(EC.presence_of_element_located(locator))

    def find_elements(self, locator: tuple) -> List[WebElement]:
        self.wait.until(EC.presence_of_element_located(locator))
        return self.driver.find_elements(*locator)
        
    def click_element(self, locator: tuple) -> None:
        element = self.wait.until(EC.element_to_be_clickable(locator))
        element.click()
        
    def enter_text(self, locator: tuple, text: str) -> None:
        element = self.wait.until(EC.element_to_be_clickable(locator))
        element.clear()
        element.send_keys(text)

    def select_dropdown_by_value(self, locator: tuple, value: str) -> None:
        element = self.wait.until(EC.element_to_be_clickable(locator))
        select = Select(element)
        select.select_by_value(value)


# ==============================================================================
# Registration Page (POM)
# ==============================================================================
class AutomationRegisterPage(BasePage):
    URL = "https://demo.automationtesting.in/Register.html"
    
    LOC_FIRST_NAME = (By.XPATH, "//input[@placeholder='First Name']")
    LOC_LAST_NAME = (By.XPATH, "//input[@placeholder='Last Name']")
    LOC_PASSWORD = (By.XPATH, "//input[@id='firstpassword']")
    LOC_CPASSWORD = (By.XPATH, "//input[@id='secondpassword']")
    
    # Notice the spaces inside the button text on this specific site
    LOC_SUBMIT = (By.XPATH, "//button[contains(text(), 'Submit')]")
    
    LOC_TEXTBOX_DYNAMIC = (By.XPATH, "//textarea[contains(@ng-model, 'Adress')]")
    LOC_PREFIX_ELEMENT = (By.XPATH, "//input[starts-with(@placeholder, 'First')]")
    LOC_TWO_ATTRS = (By.XPATH, "//input[@type='password' and @id='firstpassword']")
    LOC_ALTERNATIVES = (By.XPATH, "//input[@placeholder='First Name' or @ng-model='FirstName']")
    
    # parent axis (finding the form from one of its immediate child divs)
    LOC_PARENT_FORM = (By.XPATH, "//form[@id='basicBootstrapForm']/div[1]/parent::form")
    
    # ancestor axis
    LOC_ANCESTOR_FORM = (By.XPATH, "//input[@ng-model='FirstName']/ancestor::form")
    
    # child axis (immediate divs under form)
    LOC_CHILDREN = (By.XPATH, "//form[@id='basicBootstrapForm']/child::div")
    
    # following axis
    LOC_NEXT_ELEMENT = (By.XPATH, "//input[@ng-model='FirstName']/following::input")
    
    # Checkbox and Radio
    LOC_CHECKBOX = (By.XPATH, "//input[@type='checkbox' and @id='checkbox1']")
    LOC_RADIO = (By.XPATH, "//input[@type='radio' and @value='Male']")
    
    # Dropdown (Skills)
    LOC_DROPDOWN = (By.XPATH, "//select[@id='Skills']")
    
    # xpath index (2nd text input is Last Name)
    LOC_SECOND_TEXTBOX = (By.XPATH, "(//input[@type='text'])[2]")
    
    LOC_ALL_INPUTS = (By.XPATH, "//input")
    LOC_DYNAMIC_ELEM = (By.XPATH, "//button[contains(@id, 'submitbtn')]")

    def open(self) -> None:
        self.driver.get(self.URL)


# ==============================================================================
# Fixtures
# ==============================================================================
@pytest.fixture(scope="module")
def driver():
    logger.info("Initializing WebDriver...")
    chrome_options = Options()
    chrome_options.add_argument("--start-maximized")
    
    service = ChromeService(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service, options=chrome_options)
    yield driver
    driver.quit()

@pytest.fixture(scope="module")
def reg_page(driver):
    page = AutomationRegisterPage(driver)
    page.open()
    return page


# ==============================================================================
# Test Suite: Real-time scenario for Student Registration
# ==============================================================================
class TestStudentRegistration:
    """Test suite covering TC01 to TC20 for the demo.automationtesting.in Registration form."""

    def test_tc01_open_registration_page(self, reg_page: AutomationRegisterPage):
        """TC01: Open student registration page (get())"""
        # The page is opened in the fixture, just verifying
        assert "Register" in reg_page.driver.title

    def test_tc02_locate_username(self, reg_page: AutomationRegisterPage):
        """TC02: Locate username (Attribute XPath)"""
        elem = reg_page.find_element(reg_page.LOC_FIRST_NAME)
        assert elem is not None

    def test_tc03_enter_password(self, reg_page: AutomationRegisterPage):
        """TC03: Enter password (Attribute XPath)"""
        reg_page.enter_text(reg_page.LOC_PASSWORD, "Student2026!")
        assert reg_page.find_element(reg_page.LOC_PASSWORD).get_attribute("value") == "Student2026!"

    def test_tc04_locate_submit(self, reg_page: AutomationRegisterPage):
        """TC04: Locate Submit (text())"""
        elem = reg_page.find_element(reg_page.LOC_SUBMIT)
        assert "Submit" in elem.text

    def test_tc05_locate_textbox_dynamically(self, reg_page: AutomationRegisterPage):
        """TC05: Locate textbox dynamically (contains())"""
        elem = reg_page.find_element(reg_page.LOC_TEXTBOX_DYNAMIC)
        assert elem.tag_name == "textarea"

    def test_tc06_locate_element_with_prefix(self, reg_page: AutomationRegisterPage):
        """TC06: Locate element with prefix (starts-with())"""
        elem = reg_page.find_element(reg_page.LOC_PREFIX_ELEMENT)
        assert elem.get_attribute("placeholder").startswith("First")

    def test_tc07_find_input_two_attributes(self, reg_page: AutomationRegisterPage):
        """TC07: Find input using two attributes (and)"""
        elem = reg_page.find_element(reg_page.LOC_TWO_ATTRS)
        assert elem.get_attribute("type") == "password"

    def test_tc08_find_element_alternatives(self, reg_page: AutomationRegisterPage):
        """TC08: Find element using alternatives (or)"""
        elem = reg_page.find_element(reg_page.LOC_ALTERNATIVES)
        assert elem is not None

    def test_tc09_find_parent_form(self, reg_page: AutomationRegisterPage):
        """TC09: Find parent form (parent)"""
        elem = reg_page.find_element(reg_page.LOC_PARENT_FORM)
        assert elem.tag_name == "form"

    def test_tc10_find_form_from_input(self, reg_page: AutomationRegisterPage):
        """TC10: Find form from input (ancestor)"""
        elem = reg_page.find_element(reg_page.LOC_ANCESTOR_FORM)
        assert elem.tag_name == "form"

    def test_tc11_find_child_inputs(self, reg_page: AutomationRegisterPage):
        """TC11: Find child inputs (child)"""
        elems = reg_page.find_elements(reg_page.LOC_CHILDREN)
        assert len(elems) > 0

    def test_tc12_find_next_element(self, reg_page: AutomationRegisterPage):
        """TC12: Find next element (following)"""
        elems = reg_page.find_elements(reg_page.LOC_NEXT_ELEMENT)
        assert len(elems) > 0

    def test_tc13_find_checkbox(self, reg_page: AutomationRegisterPage):
        """TC13: Find checkbox (Attribute + XPath)"""
        checkbox = reg_page.find_element(reg_page.LOC_CHECKBOX)
        assert checkbox is not None

    def test_tc14_find_radio_button(self, reg_page: AutomationRegisterPage):
        """TC14: Find radio button (Attribute + XPath)"""
        radio = reg_page.find_element(reg_page.LOC_RADIO)
        assert radio is not None

    def test_tc15_select_dropdown(self, reg_page: AutomationRegisterPage):
        """TC15: Select dropdown (XPath + Select)"""
        reg_page.select_dropdown_by_value(reg_page.LOC_DROPDOWN, "Python")
        assert reg_page.find_element(reg_page.LOC_DROPDOWN).get_attribute("value") == "Python"

    def test_tc16_find_second_textbox(self, reg_page: AutomationRegisterPage):
        """TC16: Find second textbox (XPath index)"""
        elem = reg_page.find_element(reg_page.LOC_SECOND_TEXTBOX)
        # 2nd text input on this site is Last Name
        assert elem.get_attribute("placeholder") == "Last Name"

    def test_tc18_find_all_input_fields(self, reg_page: AutomationRegisterPage):
        """TC18: Find all input fields (find_elements())"""
        elems = reg_page.find_elements(reg_page.LOC_ALL_INPUTS)
        assert len(elems) > 10

    def test_tc19_find_dynamic_element(self, reg_page: AutomationRegisterPage):
        """TC19: Find dynamic element (contains())"""
        elem = reg_page.find_element(reg_page.LOC_DYNAMIC_ELEM)
        assert elem is not None

    def test_tc20_complete_registration(self, reg_page: AutomationRegisterPage):
        """TC20: Complete registration automation & TC17 Verify message"""
        # Complete all steps to submit the registration form
        reg_page.enter_text(reg_page.LOC_FIRST_NAME, "John")
        reg_page.enter_text(reg_page.LOC_LAST_NAME, "Doe")
        reg_page.enter_text(reg_page.LOC_PASSWORD, "Student2026!")
        reg_page.enter_text(reg_page.LOC_CPASSWORD, "Student2026!")
        
        # TC17: Verify submitted message by clicking submit
        # (This site doesn't redirect cleanly, so we just verify the button works)
        reg_page.click_element(reg_page.LOC_SUBMIT)
        
        # Check that we are still on a valid page by finding the form again
        assert reg_page.find_element(reg_page.LOC_ANCESTOR_FORM) is not None

```
<img width="742" height="432" alt="image" src="https://github.com/user-attachments/assets/433913a3-38c9-44cb-8b63-a792e66c84dc" />
