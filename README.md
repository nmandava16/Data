package com.capitalone.dealernavigator.reporting.controller;

import com.capitalone.dealernavigator.reporting.service.DealerService;
import com.capitalone.dealernavigator.reporting.model.DealerCostAndGrossResponse;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyInt;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.http.MediaType.APPLICATION_JSON;

@WebMvcTest(CostAndGrossController.class)
public class CostAndGrossControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private DealerService dealerService;

    @Test
    public void testGetCostAndGrossForLeads() throws Exception {
        int dealerId = 1;
        String month = "June";
        int year = 2023;
        int offset = 0;
        int limit = 10;

        DealerCostAndGrossResponse mockResponse = new DealerCostAndGrossResponse();
        // Populate mockResponse with test data

        given(dealerService.getDealerWithLeads(anyInt(), anyString(), anyInt(), anyInt(), anyInt()))
                .willReturn(mockResponse);

        mockMvc.perform(get("/" + dealerId + "/marketing/lead-sources")
                .param("month", month)
                .param("year", String.valueOf(year))
                .param("offset", String.valueOf(offset))
                .param("limit", String.valueOf(limit))
                .contentType(APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().json("{/* expected JSON response */}"));
    }
}
