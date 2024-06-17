package com.capitalone.dealernavigator.reporting.business;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.lenient;
import static org.mockito.Mockito.when;

import com.capitalone.dealernavigator.reporting.model.DealerCostAndGrossResponse;
import com.capitalone.dealernavigator.reporting.util.ServiceUrlBuilder;
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

@ExtendWith(MockitoExtension.class)
public class CostAndGrossControllerTest {

    @Mock
    private RestTemplate auth2RestTemplate;

    @Mock
    private ServiceUrlBuilder serviceUrlBuilder;

    @InjectMocks
    private CostAndGrossController costAndGrossController;

    private final int dealerId = 1;
    private final String month = "January";
    private final int year = 2023;
    private final int offset = 0;
    private final int limit = 10;

    @BeforeEach
    public void setup() {
        costAndGrossController = new CostAndGrossController(auth2RestTemplate, serviceUrlBuilder);
    }

    @Test
    public void testProcessDealerData_ServerError() {
        when(auth2RestTemplate.exchange(
            eq("http://localhost:8080/costandgross"),
            eq(HttpMethod.POST),
            any(HttpEntity.class),
            eq(String.class)
        )).thenThrow(new HttpServerErrorException(HttpStatus.INTERNAL_SERVER_ERROR, "Server Error"));

        Exception exception = assertThrows(HttpServerErrorException.class, () -> {
            costAndGrossController.processDealerData(dealerId, null);
        });

        String expectedMessage = "500 Server Error: [Server Error]";
        String actualMessage = exception.getMessage();

        assertTrue(actualMessage.contains(expectedMessage));
    }

    @Test
    public void testGetDealerWithLeads_Success() {
        DealerCostAndGrossResponse dealerResponse = new DealerCostAndGrossResponse();
        ResponseEntity<DealerCostAndGrossResponse> responseEntity = new ResponseEntity<>(dealerResponse, HttpStatus.OK);

        when(auth2RestTemplate.exchange(
            eq("http://localhost:8080/costandgross?month=January&year=2023&offset=0&limit=10"),
            eq(HttpMethod.GET),
            any(HttpEntity.class),
            eq(DealerCostAndGrossResponse.class)
        )).thenReturn(responseEntity);

        DealerCostAndGrossResponse response = costAndGrossController.getDealerWithLeads(dealerId, month, year, offset, limit);

        assertNotNull(response);
    }

    @Test
    public void testGetDealerWithLeads_ClientError() {
        when(auth2RestTemplate.exchange(
            eq("http://localhost:8080/costandgross?month=January&year=2023&offset=0&limit=10"),
            eq(HttpMethod.GET),
            any(HttpEntity.class),
            eq(DealerCostAndGrossResponse.class)
        )).thenThrow(new HttpClientErrorException(HttpStatus.BAD_REQUEST, "Client Error"));

        Exception exception = assertThrows(HttpClientErrorException.class, () -> {
            costAndGrossController.getDealerWithLeads(dealerId, month, year, offset, limit);
        });

        String expectedMessage = "400 Bad Request: [Client Error]";
        String actualMessage = exception.getMessage();

        assertTrue(actualMessage.contains(expectedMessage));
    }

    @Test
    public void testGetDealerWithLeads_ServerError() {
        when(auth2RestTemplate.exchange(
            eq("http://localhost:8080/costandgross?month=January&year=2023&offset=0&limit=10"),
            eq(HttpMethod.GET),
            any(HttpEntity.class),
            eq(DealerCostAndGrossResponse.class)
        )).thenThrow(new HttpServerErrorException(HttpStatus.INTERNAL_SERVER_ERROR, "Server Error"));

        Exception exception = assertThrows(HttpServerErrorException.class, () -> {
            costAndGrossController.getDealerWithLeads(dealerId, month, year, offset, limit);
        });

        String expectedMessage = "500 Server Error: [Server Error]";
        String actualMessage = exception.getMessage();

        assertTrue(actualMessage.contains(expectedMessage));
    }
}
