## Create Unit Tests

In this section, we will create a new unit test project and create a few unit tests based on behavioral mocks using MOQ.

* Add a new Unit Test project called `ReservationApi.Tests` and base it off of the xUnit .NET Core template.

* Rename the default test class to `ReservationServiceTests.cs`

* Add the following project references to the project file

  ```xml
  <ItemGroup>
    <ProjectReference Include="..\ReservationApi.Data\ReservationApi.Data.csproj" />
    <ProjectReference Include="..\ReservationApi\ReservationApi.csproj" />
  </ItemGroup>
  ```

* Add the following nuget package reference to the project file

  ```xml
    <PackageReference Include="Moq" Version="4.13.1" />
  ```

  or

  run the following command

  `dotnet add package Moq --version 4.13.1`

* Create first tests by adding the following implementation

    ```cs
    using Moq;
    using ReservationApi.Data.Intefaces;
    using ReservationApi.Data.Models;
    using ReservationApi.Services;
    using System;
    using Xunit;

    namespace ReservationApi.Tests
    {
        public class ReservationServiceTests
        {
            private Mock<IRepository<Reservation>> _mockRepo;

            public ReservationServiceTests()
            {
                _mockRepo = new Mock<IRepository<Reservation>>();
            }

            [Fact]
            public void ReservationService_Get_Reservation_Verify()
            {
                //arrange
                _mockRepo.Setup(x => x.GetAsync(It.IsAny<string>())).Verifiable();

                var reservationSvc = new ReservationService(_mockRepo.Object);

                //act
                var result = reservationSvc.GetAsync("1");

                //assert
                _mockRepo.Verify();
            }

            [Fact]
            public void ReservationService_Create_Reservation_Verify()
            {
                //arrange
                var reservation = new Reservation
                {
                    Id = Guid.NewGuid().ToString(),
                    Name = "Special Meeting",
                    RoomId = "101",
                    FromDate = DateTime.Today.ToString(),
                    ToDate = DateTime.Today.AddDays(1).ToString(),
                    Price = 1
                };

                _mockRepo.Setup(x => x.InsertAsync(It.IsAny<Reservation>())).Verifiable();

                var reservationSvc = new ReservationService(_mockRepo.Object);

                //act
                var result = reservationSvc.CreateAsync(reservation);

                //assert
                _mockRepo.Verify();
            }

            [Fact]
            public void ReservationService_Delete_Reservation_Verify()
            {
                //arrange
                var reservation = new Reservation
                {
                    Id = Guid.NewGuid().ToString(),
                    Name = "Special Meeting",
                    RoomId = "101",
                    FromDate = DateTime.Today.ToString(),
                    ToDate = DateTime.Today.AddDays(1).ToString(),
                    Price = 1
                };

                _mockRepo.Setup(x => x.DeleteAsync(It.IsAny<Reservation>())).Verifiable();

                var reservationSvc = new ReservationService(_mockRepo.Object);

                //act
                var result = reservationSvc.RemoveAsync(reservation);

                //assert
                _mockRepo.Verify();
            }

            [Fact]
            public void ReservationService_Update_Reservation_Verify()
            {
                //arrange
                var reservation = new Reservation
                {
                    Id = Guid.NewGuid().ToString(),
                    Name = "Special Meeting",
                    RoomId = "101",
                    FromDate = DateTime.Today.ToString(),
                    ToDate = DateTime.Today.AddDays(1).ToString(),
                    Price = 1
                };

                _mockRepo.Setup(x => x.UpdateAsync(It.IsAny<Reservation>())).Verifiable();

                var reservationSvc = new ReservationService(_mockRepo.Object);

                //act
                var result = reservationSvc.UpdateAsync(reservation.Id, reservation);

                //assert
                _mockRepo.Verify();
            }

        }
    }
    ```

    >**NOTE**: Because this demo doesn't contain much business logic, these tests are behavioral tests and will check to see if the method under test actually calls a specific dependency, as opposed to manipulating some state that can be asserted.
    
* Run the tests using the Test Explorer and see if they pass.