import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.mockito.quality.Strictness;
import org.mockito.junit.jupiter.MockitoSettings;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

@ExtendWith(MockitoExtension.class)
@MockitoSettings(strictness = Strictness.LENIENT)
public class CostAndGrossControllerTest {

    private MockMvc mockMvc;

    @InjectMocks
    private CostAndGrossController costAndGrossController;

    @Mock
    private DealerService dealerService;

    @Mock
    private CostAndGrossDetailsRepository costAndGrossDetailsRepository;

    @BeforeEach
    void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(costAndGrossController).build();
    }

    @Test
    void testDeleteForLead() throws Exception {
        mockMvc.perform(delete("/protected/1533244")
                .param("dealerId", "1")
                .param("LeadSourceId", "1")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isNoContent());
    }

    @Test
    void testGetForLead() throws Exception {
        mockMvc.perform(get("/1/marketing/leads/1")
                .param("month", "2024-05")
                .param("year", "2024")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isNoContent());
    }

    @Test
    void testSaveCostAndGrossForLeads() throws Exception {
        DealerCostAndGross dealerRequest = new DealerCostAndGross();
        when(dealerService.processDealerData(anyInt(), any(DealerCostAndGross.class)))
                .thenReturn(ResponseEntity.status(201).body("Created"));

        mockMvc.perform(post("/1/marketing/leads")
                .content(new ObjectMapper().writeValueAsString(dealerRequest))
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isCreated())
                .andExpect(content().string("Created"));
    }

    @Test
    void testGetCostAndGrossForLeads() throws Exception {
        DealerCostAndGrossResponse dealerResponse = new DealerCostAndGrossResponse();
        when(dealerService.getDealerWithLeads(1, "2024-05", 2024, 0, 10)).thenReturn(dealerResponse);

        mockMvc.perform(get("/1/marketing/leads")
                .param("month", "2024-05")
                .param("year", "2024")
                .param("offset", "0")
                .param("limit", "10")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk());
    }
}



@Test
public void testGetForLeadNoContent() throws Exception {
    Long dealerId = 1L, leadSourceId = 10L;
    when(dealerService.getDealerLead(dealerId, leadSourceId, "5", 2024)).thenReturn(null); // Assuming the service returns null if not found

    mockMvc.perform(get("/dealerId/marketing/leads/{dealerId}/{leadSourceId}", dealerId, leadSourceId)
            .param("month", "5")
            .param("year", "2024"))
            .andExpect(status().isNoContent());
}
