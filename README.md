import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomClientException.class)
    public ResponseEntity<String> handleCustomClientException(CustomClientException ex) {
        return new ResponseEntity<>(ex.getMessage(), ex.getStatus());
    }

    @ExceptionHandler(CustomServerException.class)
    public ResponseEntity<String> handleCustomServerException(CustomServerException ex) {
        return new ResponseEntity<>(ex.getMessage(), ex.getStatus());
    }

    @ExceptionHandler(CustomGenericException.class)
    public ResponseEntity<String> handleCustomGenericException(CustomGenericException ex) {
        return new ResponseEntity<>(ex.getMessage(), ex.getStatus());
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGenericException(Exception ex) {
        return new ResponseEntity<>("An unexpected error occurred: " + ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
    }
}











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
            throw new CustomClientException("Client error: " + e.getMessage(), e.getStatusCode());
        } catch (HttpServerErrorException e) {
            throw new CustomServerException("Server error: " + e.getMessage(), e.getStatusCode());
        } catch (Exception e) {
            throw new CustomGenericException("An unexpected error occurred: " + e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
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
            throw new CustomClientException("Client error: " + e.getMessage(), e.getStatusCode());
        } catch (HttpServerErrorException e) {
            throw new CustomServerException("Server error: " + e.getMessage(), e.getStatusCode());
        } catch (Exception e) {
            throw new CustomGenericException("An unexpected error occurred: " + e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }
}
