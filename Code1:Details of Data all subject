
#Subject Loop Cerebra ID Loop Time voxel activation Loop

# Validate required columns exist
if "Cerebra_ID" not in region_data.columns or "Region_name" not in region_data.columns:
    raise ValueError("CSV file must contain 'Cerebra_ID' and 'Region_name' columns.")

# Create mapping from Cerebra_ID to Region_name
mapping_region_id_to_name = region_data.set_index("Cerebra_ID")["Region_name"].to_dict()
# --------------------- Helper Functions ---------------------

def load_subject_video_data(subject_id, video_type):
    """
    Load brain activation data for a given subject and video type.

    Parameters:
        subject_id (str): Subject ID (e.g., 'NDARZY502FAG')
        video_type (str): Video type (e.g., "baseline", "video1", "video2", "video3")

    Returns:
        np.ndarray: Brain voxel activation data.
    """
    file_path = f"/content/drive/MyDrive/Data TU PHD DUBLIN/subjects Data/{subject_id}/evaluation/{video_type}_eLORETA.npy"
    return np.load(file_path)


def get_number_of_voxel_per_region(cerebra_id):
    """
    Get the number of voxels for a given cerebral region (Cerebra ID) and its region name.

    Parameters:
        cerebra_id (int): The ID of the cerebral region.

    Returns:
        tuple: (voxel_count (int), region_name (str))
    """
    voxel_count = np.sum(map_voxel == cerebra_id)  # Count occurrences of the Cerebra ID
    region_name = mapping_region_id_to_name.get(cerebra_id, "Unknown Region")
    return voxel_count, region_name


def plot_voxel_activation_for_region(time_location, cerebra_id, video_data):
    """
    Plot voxel activation histograms for the selected region and time location,
    for each video type in video_data.

    Parameters:
        time_location (int): The time index to analyze.
        cerebra_id (int): The Cerebra ID for the region of interest.
        video_data (dict): Dictionary of video datasets.
    """
    # Get indices for voxels in the desired region.
    region_indices = np.where(map_voxel == cerebra_id)[0]

    plt.figure(figsize=(12, 5))
    for video_label, data in video_data.items():
        # Restrict activation data to the region of interest and given time
        voxel_activation = data[region_indices, time_location]
        plt.hist(voxel_activation, bins=100, alpha=0.5, label=f"{video_label} - Time {time_location}")

    plt.title(f"Voxel Activation Distribution for Region {cerebra_id} at Time {time_location}")
    plt.xlabel("Voxel Activation Value")
    plt.ylabel("Frequency")
    plt.legend()
    plt.show()


def plot_number_of_voxel_per_region():
    """
    Plot a histogram showing the overall voxel counts per cortical region.
    """
    unique_regions = np.unique(map_voxel)
    plt.figure(figsize=(20, 5))
    plt.hist(map_voxel, bins=len(unique_regions), color="royalblue", alpha=0.7)
    plt.title("Number of Voxels per Cortical Region")
    plt.xlabel("Cortical Regions")
    plt.ylabel("Number of Voxels")
    plt.show()


# --------------------- Main Loop ---------------------

program_exit = False  # Flag to control overall program exit

while not program_exit:
    # ----- Outer Loop: Subject Selection -----
    subject_input = input("\nEnter Subject ID (or type 'exit' to quit): ").strip()
    if subject_input.lower() == "exit":
        program_exit = True
        break  # Exit the outer loop

    subject_id = subject_input  # Use the entered subject ID

    # Load video data for the subject
    try:
        video_data = {
            "Baseline": load_subject_video_data(subject_id, "baseline"),
            "Video1": load_subject_video_data(subject_id, "video1"),
            "Video2": load_subject_video_data(subject_id, "video2"),
            "Video3": load_subject_video_data(subject_id, "video3"),
        }
    except Exception as e:
        print(f"Error loading video data for subject {subject_id}: {e}")
        continue  # Ask for a new subject ID

    # Optionally, plot the overall voxel distribution per cortical region
    plot_number_of_voxel_per_region()

    # ----- Middle Loop: Cerebra ID Analysis -----
    while True:
        cerebra_input = input("\nEnter Cerebra ID (or type 'back' to choose another subject, or 'exit' to quit): ").strip()
        if cerebra_input.lower() == "back":
            print("Returning to subject selection.")
            break  # Return to the subject selection loop
        if cerebra_input.lower() == "exit":
            program_exit = True
            break  # Exit the middle loop, then outer loop

        try:
            cerebra_id = int(cerebra_input)
            voxel_count, region_name = get_number_of_voxel_per_region(cerebra_id)
            if voxel_count == 0:
                print(f"⚠️ Region '{region_name}' (ID: {cerebra_id}) has 0 voxels. Please try another Cerebra ID.")
                continue  # Remain in the Cerebra ID loop
            else:
                print(f"✅ Selected Region: {region_name} (ID: {cerebra_id}) with {voxel_count} voxels.")
        except ValueError:
            print("❌ Invalid input. Please enter a valid Cerebra ID (an integer).")
            continue  # Remain in the Cerebra ID loop

        # ----- Inner Loop: Time Location Analysis -----
        while True:
            time_input = input("\nEnter a Time Location (or type 'back' to choose another Cerebra ID, or 'exit' to quit): ").strip()
            if time_input.lower() == "back":
                print("Returning to Cerebra ID selection.")
                break  # Break inner loop to re-enter a new Cerebra ID
            if time_input.lower() == "exit":
                program_exit = True
                break  # Break inner loop, then break out to end program

            try:
                time_location = int(time_input)
                # Validate time location using the Baseline video shape
                num_timestamps = video_data["Baseline"].shape[1]
                if time_location < 0 or time_location >= num_timestamps:
                    print(f"⚠️ Invalid Time Location. Please enter a value between 0 and {num_timestamps - 1}.")
                    continue

                # Plot voxel activation histograms for the selected region and time location
                plot_voxel_activation_for_region(time_location, cerebra_id, video_data)
            except ValueError:
                print("❌ Invalid input. Please enter a valid time location (integer).")

        if program_exit:
            break  # Break out of the Cerebra ID loop if user requested to exit

    if program_exit:
        break  # Break out of the subject loop if user requested to exit

# --------------------- End of Program ---------------------
print("\nThank you for using the program. Run ended.")
