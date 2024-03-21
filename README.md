# springboot-c

public void attachStandardWorkflowToProject(String projectId, RestTemplate restTemplate) {
    String getUrl = jiraInstance.concat("/rest/api/3/workflowScheme/project")
                                .concat("?projectId=").concat(projectId);
    HttpEntity<String> request = new HttpEntity<>("", commonUtils.getHeader());
    
    try {
        ResponseEntity<JsonNode> response = restTemplate.exchange(getUrl, HttpMethod.GET, request, JsonNode.class);
        
        if (response.getStatusCode().is2xxSuccessful()) {
            JsonNode workflows = response.getBody().get("values");
            
            for (JsonNode workflow : workflows) {
                String workflowSchemeId = workflow.get("workflowScheme").get("id").asText();
                if (workflowSchemeId.equals("43136")) {
                    String workflowName = workflow.get("workflowScheme").get("defaultWorkflow").asText();
                    attachWorkflowToProject(projectId, workflowSchemeId, workflowName, restTemplate);
                    break;
                }
            }
        } else {
            System.out.println("Failed to fetch the workflow schemes with Status Code: " + response.getStatusCodeValue());
        }
    } catch (HttpClientErrorException ex) {
        System.out.println("Failed to fetch the workflow schemes with Status Code: " + ex.getStatusCode());
        log.error("Failed to fetch the workflow schemes: " + ex.getMessage() + " " + ex.getResponseBodyAsString());
    } catch (Exception e) {
        e.printStackTrace();
        log.error("Error fetching workflow schemes: " + e.getMessage());
    }
}

public void attachWorkflowToProject(String projectId, String workflowSchemeId, String workflowName, RestTemplate restTemplate) {
    String attachUrl = jiraInstance.concat("/rest/api/3/project/").concat(projectId).concat("/workflowscheme");
    HttpHeaders headers = commonUtils.getHeader();
    headers.setContentType(MediaType.APPLICATION_JSON);
    String requestBody = "{\"workflow\":\"" + workflowSchemeId + "\"}";
    HttpEntity<String> request = new HttpEntity<>(requestBody, headers);
    
    try {
        ResponseEntity<String> response = restTemplate.exchange(attachUrl, HttpMethod.PUT, request, String.class);
        
        if (response.getStatusCode().is2xxSuccessful()) {
            System.out.println("Workflow '" + workflowName + "' attached successfully to project '" + projectId + "'.");
        } else {
            System.out.println("Failed to attach workflow '" + workflowName + "' to project '" + projectId + "'. Status Code: " + response.getStatusCodeValue());
        }
    } catch (HttpClientErrorException ex) {
        System.out.println("Failed to attach workflow '" + workflowName + "' to project '" + projectId + "'. Status Code: " + ex.getStatusCode());
        log.error("Failed to attach workflow to project: " + ex.getMessage() + " " + ex.getResponseBodyAsString());
    } catch (Exception e) {
        e.printStackTrace();
        log.error("Error attaching workflow to project: " + e.getMessage());
    }
}
