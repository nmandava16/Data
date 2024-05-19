import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.http.ResponseEntity;
import org.springframework.http.HttpStatus;

public class DealerAppTests {

    @Mock
    private DealerService dealerService;

    @Mock
    private CostAndGrossDetailsRepository repository;

    @InjectMocks
    private CostAndGrossController costAndGrossController;

    private DealerCostAndGross dealerCostAndGross;
    private DealerCostAndGrossResponse dealerCostAndGrossResponse;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.initMocks(this);
        // Setup common test data
        dealerCostAndGross = new DealerCostAndGross();
        dealerCostAndGross.setDealerId(1);
        dealerCostAndGross.setMonth("January");
        dealerCostAndGross.setYear(2020);
        dealerCostAndGross.setCost(500.0);
        dealerCostAndGross.setGross(1000.0);

        dealerCostAndGrossResponse = new DealerCostAndGrossResponse();
        dealerCostAndGrossResponse.setDealerId(1);
        dealerCostAndGrossResponse.setTotalCost(500.0);
        dealerCostAndGrossResponse.setTotalGross(1000.0);
    }

    @Test
    public void testGetCostAndGrossForLeads() {
        when(dealerService.getDealerWithLeads(1L, "January", 2020, 0, 10)).thenReturn(dealerCostAndGrossResponse);
        ResponseEntity<DealerCostAndGrossResponse> response = costAndGrossController.getCostAndGrossForLeads(1L, "January", 2020, 0, 10);
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals(dealerCostAndGrossResponse, response.getBody());
    }

    @Test
    public void testSaveCostAndGrossForLeads() {
        when(dealerService.processDealerData(1, dealerCostAndGross)).thenReturn(ResponseEntity.ok().build());
        ResponseEntity<String> response = costAndGrossController.saveCostAndGrossForLeads(1, dealerCostAndGross);
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }

    @Test
    public void testProcessDealerData() {
        when(repository.save(any())).thenReturn(new DealerLeadSourceCostAndGrossEntity()); // Adjust based on actual method signatures
        dealerService.processDealerData(1, dealerCostAndGross);
        verify(repository, times(1)).save(any(DealerLeadSourceCostAndGrossEntity.class));
    }
}
