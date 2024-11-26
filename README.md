
function doPost(e) {
  // Getting the data from the form submission
  var name = e.parameter.name;
  var email = e.parameter.email;
  var message = e.parameter.message;
  
  // Open the Google Sheet and get the active sheet
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  // Append the data to the sheet
  sheet.appendRow([new Date(), name, email, message]);

  // Optional: Log submission to GitHub README (append to README.md)
  logToGitHubReadMe(name, email, message);
}

function logToGitHubReadMe(name, email, message) {
  // GitHub repository details
  var owner = 'your-github-username'; // Replace with your GitHub username
  var repo = 'your-repository'; // Replace with your repository name
  var path = 'README.md'; // Path to your README file

  // GitHub API endpoint to get the current file content
  var url = `https://api.github.com/repos/${owner}/${repo}/contents/${path}`;

  // Authorization token (create a personal access token in GitHub)
  var token = 'your-github-token'; // Replace with your GitHub token

  // Request headers including the GitHub token for authentication
  var headers = {
    'Authorization': 'token ' + token,
    'Accept': 'application/vnd.github.v3+json'
  };

  // Get the current README content (base64 encoded)
  var response = UrlFetchApp.fetch(url, {
    'method': 'get',
    'headers': headers
  });
  var data = JSON.parse(response.getContentText());
  var content = Utilities.base64Decode(data.content);
  var readmeText = Utilities.newBlob(content).getDataAsString();
  
  // Create a new log message to append
  var logMessage = `\n## New Contact Form Submission\n**Name:** ${name}\n**Email:** ${email}\n**Message:** ${message}\n---`;
  
  // Append the log message to the existing README
  readmeText += logMessage;

  // Create the new commit to push the changes
  var commitMessage = 'Updated README with new contact form submission';
  var newContent = Utilities.base64EncodeWebSafe(readmeText);
  var commitData = {
    "message": commitMessage,
    "content": newContent,
    "sha": data.sha
  };

  // Send the request to update the README file
  var updateResponse = UrlFetchApp.fetch(url, {
    'method': 'put',
    'headers': headers,
    'payload': JSON.stringify(commitData)
  });

  // Check the response
  Logger.log(updateResponse.getContentText());
}
