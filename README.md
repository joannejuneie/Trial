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
