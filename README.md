import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

public class DealerServiceTest {

    @Mock
    private CostAndGrossDetailsRepository repository; // Assuming repository has the method to fetch dealer data

    @InjectMocks
    private DealerService service;

    @BeforeEach
    public void init() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testGetDealerWithLeads() {
        // Arrange
        Long dealerId = TestData.DEALER_ID;
        String month = "May";
        Integer year = 2023;
        List<Lead> leads = TestData.LEAD_LIST; // Make sure this is initialized in your TestData class

        DealerCostAndGross expectedDealerCostAndGross = new DealerCostAndGross();
        expectedDealerCostAndGross.setDealerId(dealerId);
        expectedDealerCostAndGross.setLeadSources(leads);

        when(repository.findDealerWithLeads(dealerId, month, year)).thenReturn(expectedDealerCostAndGross);

        // Act
        DealerCostAndGross actualDealerCostAndGross = service.getDealerWithLeads(dealerId, month, year);

        // Assert
        assertNotNull(actualDealerCostAndGross, "The returned dealer cost and gross should not be null.");
        assertEquals(expectedDealerCostAndGross.getDealerId(), actualDealerCostAndGross.getDealerId(), "Dealer IDs should match.");
        assertEquals(expectedDealerCostAndGross.getLeadSources().size(), actualDealerCostAndGross.getLeadSources().size(), "Number of leads should match.");
        verify(repository, times(1)).findDealerWithLeads(dealerId, month, year);
    }
}
