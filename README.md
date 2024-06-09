import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.HttpServerErrorException;
import org.springframework.web.client.RestTemplate;

@Service
public class DealerServiceImpl implements DealerService {

    @Autowired
    @Qualifier("oAuth2RestTemplate")
    private RestTemplate oAuth2RestTemplate;

    @Autowired
    private ServiceUrlBuilder serviceUrlBuilder;

    @Override
    public ResponseEntity<String> processDealerData(int dealerId, DealerCostAndGross dealerRequest) {
        try {
            HttpHeaders headers = new HttpHeaders();
            CommonUtil.setHeaders(headers);
            HttpEntity<DealerCostAndGross> entity = new HttpEntity<>(dealerRequest, headers);

            ResponseEntity<String> response = oAuth2RestTemplate.exchange(
                serviceUrlBuilder.build(Constants.COSTANDGROSS, dealerId),
                HttpMethod.POST,
                entity,
                String.class
            );

            return new ResponseEntity<>(response.getBody(), response.getStatusCode());
        } catch (HttpClientErrorException e) {
            // Log the client error and return a specific response
            log.error("Client error occurred while processing the request: {}", e.getMessage());
            return new ResponseEntity<>("Client error: " + e.getMessage(), e.getStatusCode());
        } catch (HttpServerErrorException e) {
            // Log the server error and return a specific response
            log.error("Server error occurred while processing the request: {}", e.getMessage());
            return new ResponseEntity<>("Server error: " + e.getMessage(), e.getStatusCode());
        } catch (Exception e) {
            // Log the generic error and return a generic response
            log.error("An unexpected error occurred: {}", e.getMessage());
            return new ResponseEntity<>("An unexpected error occurred: " + e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @Override
    public DealerCostAndGrossResponse getDealerWithLeads(int dealerId, String month, Integer year, int offset, int limit) {
        try {
            HttpHeaders headers = new HttpHeaders();
            CommonUtil.setHeaders(headers);
            HttpEntity<?> entity = new HttpEntity<>(headers);

            String url = serviceUrlBuilder.build(Constants.COSTANDGROSSGET, dealerId) +
                         "?month=" + month +
                         "&year=" + year +
                         "&offset=" + offset +
                         "&limit=" + limit;

            ResponseEntity<DealerCostAndGrossResponse> response = oAuth2RestTemplate.exchange(
                url,
                HttpMethod.GET,
                entity,
                DealerCostAndGrossResponse.class
            );

            return response.getBody();
        } catch (HttpClientErrorException e) {
            // Log the client error and handle it
            log.error("Client error occurred while processing the request: {}", e.getMessage());
            // Return a response entity with status code and error message
            return new DealerCostAndGrossResponse("Client error: " + e.getMessage(), e.getStatusCode());
        } catch (HttpServerErrorException e) {
            // Log the server error and handle it
            log.error("Server error occurred while processing the request: {}", e.getMessage());
            // Return a response entity with status code and error message
            return new DealerCostAndGrossResponse("Server error: " + e.getMessage(), e.getStatusCode());
        } catch (Exception e) {
            // Log the generic error and handle it
            log.error("An unexpected error occurred: {}", e.getMessage());
            // Return a response entity with status code and error message
            return new DealerCostAndGrossResponse("An unexpected error occurred: " + e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
