package com.capitalone.dealernavigator.reporting.controller;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

import com.capitalone.dealernavigator.reporting.business.DealerService;
import com.capitalone.dealernavigator.reporting.model.DealerCostAndGrossResponse;
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
    private DealerService dealerService;

    @InjectMocks
    private CostAndGrossController costAndGrossController;

    private final int dealerId = 1;
    private final String month = "January";
    private final int year = 2023;
    private final int offset = 0;
    private final int limit = 10;

    @Test
    public void testSaveCostAndGrossForLeads_ServerError() {
        when(dealerService.processDealerData(eq(dealerId), any())).thenThrow(new HttpServerErrorException(HttpStatus.INTERNAL_SERVER_ERROR, "Server Error"));

        Exception exception = assertThrows(HttpServerErrorException.class, () -> {
            costAndGrossController.saveCostAndGrossForLeads(dealerId, null);
        });

        String expectedMessage = "500 Server Error: [Server Error]";
        String actualMessage = exception.getMessage();

        assertTrue(actualMessage.contains(expectedMessage));
    }

    @Test
    public void testGetCostAndGrossForLeads_Success() {
        DealerCostAndGrossResponse dealerResponse = new DealerCostAndGrossResponse();

        when(dealerService.getDealerWithLeads(dealerId, month, year, offset, limit)).thenReturn(dealerResponse);

        ResponseEntity<DealerCostAndGrossResponse> response = costAndGrossController.getCostAndGrossForLeads(dealerId, month, year, offset, limit);

        assertNotNull(response.getBody());
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }

    @Test
    public void testGetCostAndGrossForLeads_ClientError() {
        when(dealerService.getDealerWithLeads(dealerId, month, year, offset, limit)).thenThrow(new HttpClientErrorException(HttpStatus.BAD_REQUEST, "Client Error"));

        Exception exception = assertThrows(HttpClientErrorException.class, () -> {
            costAndGrossController.getCostAndGrossForLeads(dealerId, month, year, offset, limit);
        });

        String expectedMessage = "400 Bad Request: [Client Error]";
        String actualMessage = exception.getMessage();

        assertTrue(actualMessage.contains(expectedMessage));
    }

    @Test
    public void testGetCostAndGrossForLeads_ServerError() {
        when(dealerService.getDealerWithLeads(dealerId, month, year, offset, limit)).thenThrow(new HttpServerErrorException(HttpStatus.INTERNAL_SERVER_ERROR, "Server Error"));

        Exception exception = assertThrows(HttpServerErrorException.class, () -> {
            costAndGrossController.getCostAndGrossForLeads(dealerId, month, year, offset, limit);
        });

        String expectedMessage = "500 Server Error: [Server Error]";
        String actualMessage = exception.getMessage();

        assertTrue(actualMessage.contains(expectedMessage));
    }

    @Test
    public void testGetDealerNotification() {
        ResponseEntity<String> response = costAndGrossController.getDealerNotification(dealerId);
        
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertNull(response.getBody());
    }
}
