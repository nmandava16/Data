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
        } catch (HttpClientErrorException | HttpServerErrorException e) {
            return new ResponseEntity<>(e.getResponseBodyAsString(), e.getStatusCode());
        } catch (Exception e) {
            return new ResponseEntity<>("An error occurred while processing the request.", HttpStatus.INTERNAL_SERVER_ERROR);
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
        } catch (Exception e) {
            // Handle exception (e.g., log it and return a suitable response or rethrow it)
            return null;
        }
    }
}
