package com.example.controller;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;

import com.example.service.DealerService;
import com.example.model.DealerCostAndGrossResponse;
import com.example.model.DealerCostAndGrossRequest;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import com.fasterxml.jackson.databind.ObjectMapper;

@WebMvcTest(CostAndGrossController.class)
public class CostAndGrossControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private DealerService dealerService;

    @InjectMocks
    private CostAndGrossController costAndGrossController;

    private DealerCostAndGrossRequest dealerRequest;
    private DealerCostAndGrossResponse dealerResponse;

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
        dealerRequest = new DealerCostAndGrossRequest();
        dealerResponse = new DealerCostAndGrossResponse();
    }

    @Test
    public void testSaveCostAndGrossForLeads() throws Exception {
        when(dealerService.processDealerData(eq(1), any(DealerCostAndGrossRequest.class)))
                .thenReturn(dealerResponse);

        mockMvc.perform(post("/1/lead-sources/import-cost-gross-batch")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new ObjectMapper().writeValueAsString(dealerRequest)))
                .andExpect(status().isOk());
    }

    @Test
    public void testGetCostAndGrossForLeads() throws Exception {
        when(dealerService.getDealerWithLeads(eq(1), eq("month"), eq("year"), eq(0), eq(10)))
                .thenReturn(dealerResponse);

        mockMvc.perform(get("/1/lead-sources")
                .param("month", "month")
                .param("year", "year")
                .param("offset", "0")
                .param("limit", "10"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.dealerId").value(dealerResponse.getDealerId()));
    }

    @Test
    public void testGetDealerNotification() throws Exception {
        mockMvc.perform(get("/1/marketing/dealer-notification"))
                .andExpect(status().isOk());
    }
}
