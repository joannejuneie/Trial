using Microsoft.AspNetCore.Mvc;
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;

public class BloodTransferController : Controller
{
    private readonly HttpClient _httpClient;

    public BloodTransferController(IHttpClientFactory httpClientFactory)
    {
        _httpClient = httpClientFactory.CreateClient();
    }

    // GET: /BloodTransfer/Index
    public IActionResult Index()
    {
        return View();
    }

    // PUT: /BloodTransfer/Transfer
    [HttpPut]
    public async Task<IActionResult> Transfer([FromBody] TransferRequest request)
    {
        var response = await _httpClient.PutAsJsonAsync("api/TransferApi/Transfer", request);

        if (response.IsSuccessStatusCode)
        {
            return Ok("Transfer successful");
        }
        return BadRequest("Transfer failed");
    }

    // GET: /BloodTransfer/GetIfRequiredBottlesExist
    [HttpGet]
    public async Task<IActionResult> GetIfRequiredBottlesExist(int bloodId, int requirement)
    {
        var response = await _httpClient.GetAsync($"api/TransferApi/GetIfRequiredBottlesExist?bloodId={bloodId}&requirement={requirement}");

        if (response.IsSuccessStatusCode)
        {
            var message = await response.Content.ReadAsStringAsync();
            return Ok(message);
        }
        return NotFound("Not enough bottles");
    }
}







/::::::::::::::

@{
    ViewData["Title"] = "Blood Transfer";
}

<h2>Blood Transfer</h2>

<div>
    <form id="transferForm">
        <div>
            <label for="hospitalName">Hospital Name:</label>
            <input type="text" id="hospitalName" name="hospitalName" required />
        </div>
        <div>
            <label for="bottlesRequired">Number of Bottles Required:</label>
            <input type="number" id="bottlesRequired" name="bottlesRequired" required />
        </div>
        <div>
            <label for="bloodGroup">Blood Group:</label>
            <input type="text" id="bloodGroup" name="bloodGroup" required />
        </div>
        <div>
            <button type="button" id="checkAvailability">Check Availability</button>
            <button type="button" id="transfer" disabled>Transfer</button>
        </div>
    </form>
</div>

@section Scripts {
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script>
        $(document).ready(function () {
            $('#checkAvailability').on('click', function () {
                var bloodGroup = $('#bloodGroup').val();
                var bottlesRequired = $('#bottlesRequired').val();

                // Assuming you have a way to get bloodId from bloodGroup
                var bloodId = getBloodIdFromGroup(bloodGroup);

                $.ajax({
                    url: '/BloodTransfer/GetIfRequiredBottlesExist',
                    type: 'GET',
                    data: { bloodId: bloodId, requirement: bottlesRequired },
                    success: function (response) {
                        alert(response);
                        $('#transfer').prop('disabled', false);
                    },
                    error: function (response) {
                        alert(response.responseText);
                        $('#transfer').prop('disabled', true);
                    }
                });
            });

            $('#transfer').on('click', function () {
                var hospitalName = $('#hospitalName').val();
                var bottlesRequired = $('#bottlesRequired').val();
                var bloodGroup = $('#bloodGroup').val();

                var requestData = {
                    BloodGroup: bloodGroup,
                    Requirement: bottlesRequired,
                    HospitalName: hospitalName
                };

                $.ajax({
                    url: '/BloodTransfer/Transfer',
                    type: 'PUT',
                    contentType: 'application/json',
                    data: JSON.stringify(requestData),
                    success: function (response) {
                        alert(response);
                    },
                    error: function (response) {
                        alert(response.responseText);
                    }
                });
            });

            function getBloodIdFromGroup(bloodGroup) {
                // Implement this function to return the bloodId based on the bloodGroup
                // This could involve another AJAX call to fetch the bloodId from the server
                return 1; // Placeholder value
            }
        });
    </script>
}













/:/:::;::::;;;((
public class BloodTransferController : Controller
{
    private readonly HttpClient _httpClient;

    public BloodTransferController(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    // PUT: /Transfer
    [Route("Transfer")]
    [HttpPut]
    public async Task<IActionResult> PutBloodInventoryBL(string bloodGroup, int requirement, string hospitalName)
    {
        var requestData = new
        {
            BloodGroup = bloodGroup,
            Requirement = requirement,
            HospitalName = hospitalName
        };

        var response = await _httpClient.PutAsJsonAsync("api/TransferApi/Transfer", requestData);

        if (response.IsSuccessStatusCode)
        {
            return Ok("Transfer successful");
        }
        return BadRequest("Transfer failed");
    }

    // GET: /GetIfRequiredBottlesExist
    [Route("GetIfRequiredBottlesExist")]
    [HttpGet]
    public async Task<IActionResult> GetIfRequiredBottlesExist(int bloodId, int requirement)
    {
        var response = await _httpClient.GetAsync($"api/TransferApi/GetIfRequiredBottlesExist?bloodId={bloodId}&requirement={requirement}");

        if (response.IsSuccessStatusCode)
        {
            var message = await response.Content.ReadAsStringAsync();
            return Ok(message);
        }
        return NotFound("Not enough bottles");
    }
}







using BBMS_BL;
using BBMS_Entities;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace BBMS_Core_Web_API.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class BloodTransferController : ControllerBase
    {

        //PUT: /Transfer
        [Route("Transfer")]
        [HttpPut]
        //[ResponseType(typeof(BloodInventory))]
        public BloodBankManagementSystemHttpStatusCode PutBloodInventoryBL(string bloodGroup, int requirement)
        {
            //write validation for blood group and requirement in bl
            BloodTransferBL inventory = new BloodTransferBL();

            return inventory.UpdateBloodInventoryBL(bloodGroup, requirement);
        }


        // GET: /GetIfRequiredBottlesExist
        [Route("GetIfRequiredBottlesExist")]
        [HttpGet]
        // [ResponseType(typeof(BloodInventory))]

        public async Task<ActionResult> GetIfRequiredBottlesExist(int bloodId, int requirement)
        {
            BloodTransferBL inventory = new BloodTransferBL();
            if (inventory.GetIfRequiredBottlesExistBL(bloodId, requirement) >= requirement)
            {
                return Ok("Bottles exist for transfer");
            }
            return NotFound();

        }

    }
}
