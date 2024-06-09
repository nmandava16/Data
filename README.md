import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.when;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.HttpClientErrorException;
import org.springframework.web.client.HttpServerErrorException;
import org.springframework.web.client.RestTemplate;

import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
public class DealerServiceImplTest {

    @Mock
    private RestTemplate oAuth2RestTemplate;

    @Mock
    private ServiceUrlBuilder serviceUrlBuilder;

    @InjectMocks
    private DealerServiceImpl dealerService;

    private final int dealerId = 1;
    private final String month = "January";
    private final int year = 2023;
    private final int offset = 0;
    private final int limit = 10;
    private final DealerCostAndGross dealerRequest = new DealerCostAndGross();
    private final DealerCostAndGrossResponse dealerResponse = new DealerCostAndGrossResponse();

    @BeforeEach
    public void setUp() {
        when(serviceUrlBuilder.build(Constants.COSTANDGROSS, dealerId)).thenReturn("http://localhost:8080/costandgross");
        when(serviceUrlBuilder.build(Constants.COSTANDGROSSGET, dealerId)).thenReturn("http://localhost:8080/costandgrossget");
    }

    @Test
    public void testProcessDealerData_Success() {
        when(oAuth2RestTemplate.exchange(
                eq("http://localhost:8080/costandgross"),
                eq(HttpMethod.POST),
                any(HttpEntity.class),
                eq(String.class)
        )).thenReturn(new ResponseEntity<>("Success", HttpStatus.OK));

        ResponseEntity<String> response = dealerService.processDealerData(dealerId, dealerRequest);
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals("Success", response.getBody());
    }

    @Test
    public void testProcessDealerData_ClientError() {
        when(oAuth2RestTemplate.exchange(
                eq("http://localhost:8080/costandgross"),
                eq(HttpMethod.POST),
                any(HttpEntity.class),
                eq(String.class)
        )).thenThrow(new HttpClientErrorException(HttpStatus.BAD_REQUEST, "Client Error"));

        ResponseEntity<String> response = dealerService.processDealerData(dealerId, dealerRequest);
        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        assertEquals("Client Error", response.getBody());
    }

    @Test
    public void testProcessDealerData_ServerError() {
        when(oAuth2RestTemplate.exchange(
                eq("http://localhost:8080/costandgross"),
                eq(HttpMethod.POST),
                any(HttpEntity.class),
                eq(String.class)
        )).thenThrow(new HttpServerErrorException(HttpStatus.INTERNAL_SERVER_ERROR, "Server Error"));

        ResponseEntity<String> response = dealerService.processDealerData(dealerId, dealerRequest);
        assertEquals(HttpStatus.INTERNAL_SERVER_ERROR, response.getStatusCode());
        assertEquals("An error occurred while processing the request.", response.getBody());
    }

    @Test
    public void testGetDealerWithLeads_Success() {
        when(oAuth2RestTemplate.exchange(
                eq("http://localhost:8080/costandgrossget?month=January&year=2023&offset=0&limit=10"),
                eq(HttpMethod.GET),
                any(HttpEntity.class),
                eq(DealerCostAndGrossResponse.class)
        )).thenReturn(new ResponseEntity<>(dealerResponse, HttpStatus.OK));

        DealerCostAndGrossResponse response = dealerService.getDealerWithLeads(dealerId, month, year, offset, limit);
        assertNotNull(response);
    }

    @Test
    public void testGetDealerWithLeads_ClientError() {
        when(oAuth2RestTemplate.exchange(
                eq("http://localhost:8080/costandgrossget?month=January&year=2023&offset=0&limit=10"),
                eq(HttpMethod.GET),
                any(HttpEntity.class),
                eq(DealerCostAndGrossResponse.class)
        )).thenThrow(new HttpClientErrorException(HttpStatus.BAD_REQUEST, "Client Error"));

        DealerCostAndGrossResponse response = dealerService.getDealerWithLeads(dealerId, month, year, offset, limit);
        assertNull(response);
    }

    @Test
    public void testGetDealerWithLeads_ServerError() {
        when(oAuth2RestTemplate.exchange(
                eq("http://localhost:8080/costandgrossget?month=January&year=2023&offset=0&limit=10"),
                eq(HttpMethod.GET),
                any(HttpEntity.class),
                eq(DealerCostAndGrossResponse.class)
        )).thenThrow(new HttpServerErrorException(HttpStatus.INTERNAL_SERVER_ERROR, "Server Error"));

        DealerCostAndGrossResponse response = dealerService.getDealerWithLeads(dealerId, month, year, offset, limit);
        assertNull(response);
    }
}
