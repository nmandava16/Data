import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*;
import static org.mockito.Mockito.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

@WebMvcTest(CostAndGrossController.class)
public class CostAndGrossControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private DealerService dealerService;  // Assuming DealerService is used in your controller

    @BeforeEach
    void setUp() {
        // Mock setup can be done here if needed for each test
    }

    @Test
    void testGetCostAndGrossForLeads() throws Exception {
        int dealerId = 1;
        String month = "May";
        int year = 2023;
        int offset = 0;
        int limit = 10;

        DealerCostAndGrossResponse mockResponse = new DealerCostAndGrossResponse();
        // setup your mock response

        when(dealerService.getDealerWithLeads(dealerId, month, year, offset, limit))
            .thenReturn(mockResponse);

        mockMvc.perform(get("/dealer/{dealerId}/marketing/leads", dealerId)
                .param("month", month)
                .param("year", String.valueOf(year))
                .param("offset", String.valueOf(offset))
                .param("limit", String.valueOf(limit))
                .accept(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.someField").value("someValue"));  // Adjust according to actual JSON response structure

        verify(dealerService).getDealerWithLeads(dealerId, month, year, offset, limit);
    }
}
        
