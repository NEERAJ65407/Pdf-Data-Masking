# Pdf-Data-Masking 

public void selectLastOptionForAllSecurityQuestions() {
    // Wait setup (optional for reliability)
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

    // Get all fieldsets (each one wraps a security question)
    List<WebElement> questionFieldsets = driver.findElements(By.xpath("//fieldset"));

    for (WebElement question : questionFieldsets) {
        // Find all radio answer divs inside this question
        List<WebElement> answerOptions = question.findElements(By.xpath(".//div[contains(@class, 'radio verid-answer')]"));

        if (!answerOptions.isEmpty()) {
            // Get the last option
            WebElement lastOption = answerOptions.get(answerOptions.size() - 1);

            // Click the label (triggers the input selection)
            WebElement label = lastOption.findElement(By.tagName("label"));

            // Optional: wait until clickable before clicking
            wait.until(ExpectedConditions.elementToBeClickable(label)).click();
        }
    }
}
