# Pdf-Data-Masking 

"""
=========================
SECTION 1: Feature (Gherkin)
=========================
File: features/lxs_apply_regression.feature
"""

feature_text = """
@LxsCloTokenE2EDeclineFlow
Scenario: Lxs token E2E declined flow
  Given Customer logs in to the portal with declined CPC token
  Then user can see Desired Loan Amount text box and drop downs for Loan Purpose and Gross Annual Income text box and drop downs for Source of Income and Residence Status
  When user enters value of Desired Loan Amount and Loan Purpose and Gross Annual Income Source of Income and Residence Status
  Then user should continue after selecting any of these options
  Then user should see application declined message
"""

"""
=========================
SECTION 2: Page Object
=========================
File: pages/lxs_apply_loan_page.py
"""

from playwright.async_api import Page, expect

class LXSApplyLoanPage:
    def __init__(self, page: Page):
        self.page = page
        # Locators
        self.desired_loan_amount = page.locator("//input[@id='desired-loan-amount']")
        self.loan_purpose_dropdown = page.locator("//div[@class='cl-dropdown-header']//span[text()='Please select']")
        self.gross_annual_income = page.locator("//input[@id='gross-annual-income-input']")
        self.source_of_income_dropdown = page.locator("(//div[@class='cl-dropdown-header']//span[text()='Please select'])[2]")
        self.residence_status_dropdown = page.locator("(//div[@class='cl-dropdown-header']//span[text()='Please select'])[3]")
        self.residence_value_other = page.locator("//span[text()='Other']")
        self.continue_button = page.locator("//button[contains(text(),'Continue')]")
        self.declined_message = page.locator("//div[@class='declined-message']")

    # -------- Actions --------
    async def navigate_with_declined_token(self, token: str):
        await self.page.goto(f"/clapply/loanoffer?token={token}")

    async def verify_fields_visible(self):
        await expect(self.desired_loan_amount).to_be_visible()
        await expect(self.loan_purpose_dropdown).to_be_visible()
        await expect(self.gross_annual_income).to_be_visible()
        await expect(self.source_of_income_dropdown).to_be_visible()
        await expect(self.residence_status_dropdown).to_be_visible()

    async def enter_offer_page_details(self, amount: str, purpose: str, income: str):
        await self.desired_loan_amount.fill(amount)
        await self.loan_purpose_dropdown.click()
        await self.page.locator(f"//span[text()='{purpose}']").click()
        await self.gross_annual_income.fill(income)
        await self.source_of_income_dropdown.click()
        await self.page.locator("//span[text()='Employment']").click()
        await self.residence_status_dropdown.click()
        await self.residence_value_other.click()

    async def click_continue(self):
        await self.continue_button.click()

    async def verify_declined_message(self):
        await expect(self.declined_message).to_have_text(
            "Sorry, but we’re unable to approve your application for a Barclays Personal Loan at this time."
        )

"""
=========================
SECTION 3: Step Definitions
=========================
File: step_definitions/lxs_apply_loan_steps.py
"""

from pytest_bdd import given, when, then

@given("Customer logs in to the portal with declined CPC token")
async def login_declined(page):
    loan_page = LXSApplyLoanPage(page)
    await loan_page.navigate_with_declined_token("DECLINED_TOKEN")

@then("user can see Desired Loan Amount text box and drop downs for Loan Purpose and Gross Annual Income text box and drop downs for Source of Income and Residence Status")
async def verify_fields(page):
    loan_page = LXSApplyLoanPage(page)
    await loan_page.verify_fields_visible()

@when("user enters value of Desired Loan Amount and Loan Purpose and Gross Annual Income Source of Income and Residence Status")
async def enter_details(page):
    loan_page = LXSApplyLoanPage(page)
    await loan_page.enter_offer_page_details("7000", "Personal Loan", "50000")

@then("user should continue after selecting any of these options")
async def continue_flow(page):
    loan_page = LXSApplyLoanPage(page)
    await loan_page.click_continue()

@then("user should see application declined message")
async def declined_message(page):
    loan_page = LXSApplyLoanPage(page)
    await loan_page.verify_declined_message()

"""
=========================
SECTION 4: Test Runner
=========================
File: tests/test_lxs_regression.py
"""

from pytest_bdd import scenarios

scenarios("../features/lxs_apply_regression.feature")

"""
=========================
SECTION 5: Playwright Fixtures
=========================
File: conftest.py
"""

import pytest
from playwright.async_api import async_playwright

@pytest.fixture(scope="session")
async def browser():
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=False)
        yield browser
        await browser.close()

@pytest.fixture
async def page(browser):
    context = await browser.new_context()
    page = await context.new_page()
    yield page
    await page.close()
